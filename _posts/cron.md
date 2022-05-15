```shell
docker build --platform amd64 --build-arg DEPENDENCY=build/dependency --tag make_plot:latest .
```

```shell
docker tag d5dfee69af44 wjdqlsdlsp/make_plot
```

```shell
docker push wjdqlsdlsp/make_plot
```

```shell
apiVersion: batch/v1
kind: CronJob
metadata:
  name: semogong-dash
spec:
  schedule: "* 4 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: semogong-dash
            image: wjdqlsdlsp/make_plot:latest
            envFrom:
            - configMapRef:
                name: config-dev
          restartPolicy: OnFailure
```

