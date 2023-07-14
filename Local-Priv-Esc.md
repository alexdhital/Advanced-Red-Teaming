# PowerUp

### Get services with unquoted paths and spaces in their name
```
Get-ServiceUnquoted -Verbose
```
### Get services where current user can write to its binary path or change arguments to the binary
```
Get-ModifiableServiceFile -Verbose
```
### Get the services whose configuration current user can modify
```
Get-ModifiableService -Verbose
```