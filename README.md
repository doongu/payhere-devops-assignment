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

```
'''
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
```



## 2.

## 3.

## 4.

## 5.

## 6.

## 7.

## 8.

## 9.

## 10.

## 11.
