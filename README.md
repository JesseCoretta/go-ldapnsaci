# go-ldapnsaci

Package aci implements the entirety of the Netscape ACIv3 syntax, used in multiple directory products such as Oracle Directory, 389DS, OpenDJ, et al., for the purpose of defining access privileges.

Please note that each implementation of Netscape's ACIv3 syntax has subtle variations.  While this library aims to be a _complete_ implementation of the syntax, your directory system may or may not support all desired features. Review your vendor documentation to determine which features are available to you.

## Features

 - ***Thoroughly*** documented with a crazy number of examples
 - Extreme test coverage
 - Vendor agnostic design (_all_ possible ACIv3 features are implemented)
 - BindRule parenthetical preservation (will observe use, or avoidance, of parenthetical statements)
 - Padding preservation (e.g.: `( userdn = "ldap:///anyone" )` vs. `(userdn="ldap:///anyone")`)
 - Panic-proof indexing for multi-valued statement (e.g.: bind rules)
 - Supports creation of ACIv3 statements by text parsing or manual type instance assembly

## Parse Example

```go
	// Define a raw ACIv3 statement
	raw := `( targetfilter = "(&(objectClass=employee)(objectClass=engineering))" )( targetcontrol = "1.2.3.4" || "1.2.3.5" )( targetscope = "onelevel" )(version 3.0; acl "Allow read and write for anyone using greater than or equal 128 SSF - extra nesting"; allow(read,write) ( ( ( userdn = "ldap:///anyone" ) AND ( ssf >= "71" ) ) AND NOT ( dayofweek = "Wed" OR dayofweek = "Fri" ) ); )`

	// Parse raw statement
	i, err := NewInstruction(raw)
	if err != nil {
		fmt.Println(err)
		return
	}

	// We now have an object -- i -- that contains the
	// complete statement above, but as a structured type
	// instance that can be examined and traversed.

	// Call the first PermissionBindRule index and
	// print the rights statement.
	perm := i.PB.Index(0).Permission()
	fmt.Printf("Permission: %s\n", perm)

	// Returns: allow(read,write)

	// Call the first BindRule, and then call the
	// first sub-statement. Print it.
	rule := i.PB.Index(0).BindRule().Index(0).Index(0)
	fmt.Printf("BindRule: %s\n", rule)

	// Returns (userdn="ldap:///anyone")
```
