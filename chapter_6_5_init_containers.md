# Init Containers

## Example 

```yaml
spec:
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'echo The app is starting!']
  containers:
  - name: myservice
    image: myservice
```
