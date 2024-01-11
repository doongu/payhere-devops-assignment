# payhere-devops-assignment
페이히어 데브옵스 엔지니어 과제 제출합니다.

## 1.
<br/>
gradle을 설치하고 아래 명령어를 통해 gradle 환경으로 바꾸었습니다.
이후에는 maven환경을 삭제 해주었습니다.

```
// gradle 버전 : gradle-7.4.2
gradle init
```
<br/>
아래 명령어를 통해 jar파일을 만들어 주었습니다.

```
gradleb build
```

<br/>

이후에는 Dockerfile을 작성해 image를 생성해주고 제 docker hub으로 올렸습니다. 

Dockerfile : https://github.com/doongu/spring-petclinic-data-jdbc/blob/master/Dockerfile<br/>
docker-hub : https://hub.docker.com/repository/docker/doongu/petclinic-spring/general



<br/>
Database는 아래 명령어를 통해 도커 이미지를 빌드 했습니다.

```
docker compose up
```
<br/>
k8s환경에서는 container image를 지정해야 해서 기존 compose파일 환경에 맞게 yaml을 구성해주었습니다. (kompose를 통해 변환하고 참조해 yaml을 작성했습니다.)


```yaml
...
    spec:
      containers:
      - name: mysql
        image: mysql:8.0.26
        env:
          - name: MYSQL_DATABASE
            value: petclinic
          - name: MYSQL_ROOT_PASSWORD
            value: petclinic
        ports:
        - containerPort: 3306
...
```

## 2.
<br/>
어플리케이션의 로그를 저장하기 위해 logback-spring.xml을 추가해주었습니다.<br/>

logback-spring.xml : https://github.com/doongu/spring-petclinic-data-jdbc/blob/master/src/main/resources/logback-spring.xml<br/>

<br/>
이후 컨테이너에 접속해 petclinic.log가 존재하는 것을 확인했고, k8s에선 host에 저장하기 위해 hostpath를 통해 마운트를 진행했습니다.

```yaml
...
        volumeMounts:
        - name: log-volume
          mountPath: /doongu/logs
      volumes:
      - name: log-volume
        hostPath:
          type: Directory
          path: /logs
...
```

<br/>
결과 사진


## 3.

healthcheck api를 제작했습니다.

healthcheck api : https://github.com/doongu/spring-petclinic-data-jdbc/blob/master/src/main/java/org/springframework/samples/petclinic/health/HealthController.java 

<br/>
10초에 한 번씩 확인하기 위해 livenessProbe 설정을 아래와 같이 해주었습니다.

```yaml
...
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 10
...
```

## 4.
terminationGracePeriodSeconds 옵션을 추가해 30초 이내에 종료되지 않으면 SIGKILL이 되도록 하였습니다.

```yaml
...
      terminationGracePeriodSeconds: 30
...
```

## 5.

Deployment에서 RollingUpdate시에 max surge, max unavailable와 replicas를 적절히 설정해주었습니다.
```yaml
...
kind: Deployment
metadata:
  name: petclinic-deployment
  namespace: default
spec:
  replicas: 4
...
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%

```

<br/>
이후로 scale-in, scale-out을 위해 HPA구성을 아래와 같이 했습니다.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: petclinic-deployment
  minReplicas: 1
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
```

## 6.
Dockerfile 설정을 아래와 같이 해주었습니다.
```Dockerfile
...
RUN useradd -r -u 999 doongu
RUN mkdir -p /doongu
RUN chown doongu /doongu
...
```


## 7.



## 8.
mysql을 sevice로 배포하고, cluster domain인 mysql.default.svc.cluster.local에 connection하도록 application.properties를 아래와 같이 수정했습니다.

application.properties : https://github.com/doongu/spring-petclinic-data-jdbc/blob/master/src/main/resources/application.properties
```properties
...
spring.datasource.url=jdbc:mysql://mysql.default.svc.cluster.local/petclinic
...
```

## 9.
nginx ingress-controller를 통해 외부에서 접근할 수 있도록 했습니다.

결과 사진

## 10.
별도의 namespace를 지정하지 않고 진행했습니다.


## 11.
실행방법
