

You can delete an entire package our from the NSO by deleting the package from the `packages` dir followed by a `packages reload` . This needs to be forced for the deletion .


```sh
admin@ncs# packages reload
Error: The following namespaces will be deleted by upgrade:
l2vpn: http://com/example/l2vpn
If this is intended, proceed with 'force' parameter.
admin@ncs# packages reload force

>>> System upgrade is starting.
>>> Sessions in configure mode must exit to operational mode.
>>> No configuration changes can be performed until upgrade has completed.
>>> System upgrade has completed successfully.
reload-result {
    package cisco-ios
    result true
}
reload-result {
    package cisco-iosxr
    result true
}
```
