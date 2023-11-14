# Go hello world
## Go hello_world
1. Initiate go project
    ```
    C:\Users\ASUS\Desktop\samsung_assignment>go mod init hello_world
    go: creating new go.mod: module hello_world
    ```

2. install gin 
    ```
    C:\Users\ASUS\Desktop\samsung_assignment>go get github.com/gin-gonic/gin
    go: added github.com/bytedance/sonic v1.9.1
    go: added github.com/chenzhuoyu/base64x v0.0.0-20221115062448-fe3a3abad311
    go: added github.com/gabriel-vasile/mimetype v1.4.2
    go: added github.com/gin-contrib/sse v0.1.0
    go: added github.com/gin-gonic/gin v1.9.1
    go: added github.com/go-playground/locales v0.14.1
    go: added github.com/go-playground/universal-translator v0.18.1
    ....
    ```
3. Check if go.mod and go.sum is exists as it is required to build docker file later
    ```
    PS C:\Users\ASUS\Desktop\samsung_assignment> dir
        Directory: C:\Users\ASUS\Desktop\samsung_assignment
    Mode                 LastWriteTime         Length Name
    ----                 -------------         ------ ----
    -a----        11/12/2023  11:44 PM           1301 go.mod
    -a----        11/12/2023  11:44 PM           7076 go.sum
    ```
4. hello_world code
    ```
    package main
    
    import (
    	"net/http"
    	"github.com/gin-gonic/gin"
    )//importing gin as web frame work and net.http for basic http function such as http status code 200
    
    func main() {
    	router := gin.Default()//initialize gin 
    
    	router.GET("/", func(c *gin.Context) {
    		c.String(http.StatusOK, "Hello World")
    	})//create router to return hello world
    
    	router.Run()//run the server
    }
    ```


## Docker

1. Dockerfile
    ```
    #using the golang base image
    FROM golang:latest
    
    #Working on /app directory, first copy go mod and sum so that change in code
    #doesnot result in redownloading dependecies
    WORKDIR /app
    COPY go.mod .
    COPY go.sum .
    #run dependencies
    RUN go mod download
    
    #copy remaining of the file and build the backend file
    COPY . .
    RUN go build -o main
    CMD "./main"
    EXPOSE 8080
    ```
    

2. Docker build command:
building a docker image from the existsing go file and dockerfile with label dumac/hello_world with tag latest
    ```
    docker build -t dumac/hello_world docker .
    ```


3. Docker image build Result:
    ```
    [+] Building 28.0s (13/13) FINISHED
     => [internal] load build definition from Dockerfile                                                               0.1s
     => => transferring dockerfile: 199B                                                                               0.0s
     => [internal] load .dockerignore                                                                                  0.0s
     => => transferring context: 2B                                                                                    0.0s
     => [internal] load metadata for docker.io/library/golang:latest                                                   2.3s
     => [auth] library/golang:pull token for registry-1.docker.io                                                      0.0s
     => [internal] load build context                                                                                  0.1s
     => => transferring context: 431B                                                                                  0.0s
     => [1/7] FROM docker.io/library/golang:latest@sha256:81cd210ae58a6529d832af2892db822b30d84f817a671b8e1c15cff0b27  0.0s
     => CACHED [2/7] WORKDIR /app                                                                                      0.0s
     => CACHED [3/7] COPY go.mod .                                                                                     0.0s
     => CACHED [4/7] COPY go.sum .                                                                                     0.0s
     => CACHED [5/7] RUN go mod download                                                                               0.0s
     => [6/7] COPY . .                                                                                                 0.1s
     => [7/7] RUN go build -o main                                                                                    23.8s
     => exporting to image                                                                                             1.5s
     => => exporting layers                                                                                            1.4s
     => => writing image sha256:207516a3844f7d5ef5bc340743ebcef5aec0f94b0b649c600cfaae8a59bdaea5                       0.0s
     => => naming to docker.io/dumac/hello_world                                                                       0.0s
    ```
4. Check available docker images to make sure image is built:

    ```
    PS C:\Users\ASUS\Desktop\samsung_assignment> docker images
    REPOSITORY                       TAG       IMAGE ID       CREATED         SIZE
    dumac/hello_world                latest    207516a3844f   4 minutes ago   1.15GB
    ```


5. change tag according to repo name 
    ```
     PS C:\Users\ASUS\Desktop\samsung_assignment> docker tag dumac/hello_world dumacrevano2000/go_hello_world:v1
    ```
6. docker push image
    ```
    PS C:\Users\ASUS\desktop\samsung_assignment> docker push dumacrevano2000/go_hello_world:v1
    The push refers to repository [docker.io/dumacrevano2000/go_hello_world]
    5ffbbd1ec49d: Pushed
    7e2397ce792b: Pushed
    360b2f660f3a: Pushed
    3a6e449aa335: Pushed
    a540798760e6: Pushed
    cc4db2bd96ac: Pushed
    64834ad7447a: Mounted from library/golang
    ae05420f6985: Mounted from library/golang
    dfe25755ef07: Mounted from library/golang
    266def75d28e: Mounted from library/golang
    29e49b59edda: Mounted from library/golang
    1777ac7d307b: Mounted from library/golang
    v1: digest: sha256:c38cf5f5206bd372d8c8a63b895d1e134458d5bf16b8ab9181c7c54445873353 size: 2838
    ```

7. docker-compose.yml

        #Make go Web Server
        version: '3'
        services:
          go-app:
            image : 'dumacrevano2000/go_hello_world:v1'
            ports: 
             - "1080:8080"

8. run docker-compose up 
    ```
    PS C:\Users\ASUS\desktop\samsung_assignment> docker-compose up
    samsung_assignment_go-app_1 is up-to-date
    Attaching to samsung_assignment_go-app_1
    ....    
    ....
    go-app_1  | [GIN-debug] Environment variable PORT is undefined. Using port :8080 by default
    go-app_1  | [GIN-debug] Listening and serving HTTP on :8080
    ```
9. curl 127.0.0.1:1080  to make sure docker docker container is working
    ```
    C:\Users\ASUS>curl -XGET 127.0.0.1:1080
    Hello World
    ```

## Kubernetes

### Deploy App in Kubernetes
1. Create Secret that would be used as reference for docker 
    ```
    C:\Users\ASUS>kubectl create secret docker-registry my-secret --docker-server=https://index.docker.io/v1/ --docker-username=XXXXX --docker-password=XXXXX --docker-email=XXXXX
    
    secret/my-secret created
    ```

2. Construct the deployment.yaml
    ```
    apiVersion: apps/v1
    kind: Deployment #deployment is for stateless app  like webserver
    metadata:
      name: hello-world
    spec:
      replicas: 2 # 2 pods is used in this deployment
      selector:
        matchLabels:
          app: go-hello
      template:
        metadata:
          labels:
            app: go-hello
        spec:
          containers:
          - name: go-hello-world
            image: dumacrevano2000/go_hello_world:v1 # image from previously pushed image
          imagePullSecrets:
          - name: my-secret #REFER to the secret we created before
    ```
3. Construct service.yaml
    ```
    apiVersion: v1
    kind: Service
    metadata:
      name: hello-world
    spec:
      selector:
        app: go-hello # deployment label need to match
      ports:
        - protocol: TCP
          port: 8080  # Port on the service
          targetPort: 8080  # Port on the pod
      type: NodePort
    ```

4. Kubectl apply deployment and service
    ```
    C:\Users\ASUS\Desktop\samsung_assignment>kubectl apply -f deployment.yaml
    deployment.apps/hello-world created
    
    C:\Users\ASUS\Desktop\samsung_assignment>kubectl apply -f service.yaml
    service/hello-world created
    ```

5. forward service connection to host network to access from local browser
    ```
    C:\Users\ASUS\Desktop>kubectl port-forward service/hello-world 7080:8080
    Forwarding from 127.0.0.1:7080 -> 8080
    Forwarding from [::1]:7080 -> 8080
    Handling connection for 7080
    Handling connection for 7080
    ```

### List pods and services

1. Kubectl get service, deployment and pods
    ```
    C:\Users\ASUS\Desktop\samsung_assignment>kubectl get services
    NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
    hello-world   NodePort    10.111.200.53   <none>        8080:32015/TCP   2m24s
    kubernetes    ClusterIP   10.96.0.1       <none>        443/TCP          21h
    
    C:\Users\ASUS\Desktop\samsung_assignment>kubectl get deployment
    NAME          READY   UP-TO-DATE   AVAILABLE   AGE
    hello-world   2/2     2            2           5m17s
    
    C:\Users\ASUS\Desktop\samsung_assignment>kubectl get pod
    NAME                           READY   STATUS    RESTARTS   AGE
    hello-world-5667c65b96-kdlqc   1/1     Running   0          5m35s
    hello-world-5667c65b96-wtwhq   1/1     Running   0          5m35s
    
    ``` 




### Access your deployed App

1. curl to make sure if go hello world web is accessible from host network
    ```
    C:\Users\ASUS\Desktop\samsung_assignment>curl localhost:7080
    Hello World
    ```



## AWS 
1. Create Key pair for the later login
    ```
    aws ec2 create-key-pair --key-name test_key1 --output json > test_key1.pem
    ```
2. Create key pairs for ssh use later
    ```
    C:\Users\ASUS\Desktop\test_aws_cli>aws ec2 create-key-pair --key-name test_key1 --output json > test_key1.pem
    
    C:\Users\ASUS\Desktop\test_aws_cli>aws ec2 describe-key-pairs --output json
    {
        "KeyPairs": [
            {
                "KeyPairId": "key-0017553d3997680b2",
                "KeyFingerprint": "96:fd:9f:8a:d3:f6:f1:3f:94:c3:fd:7e:72:ec:fd:90:9a:2a:0c:38",
                "KeyName": "test_key1",
                "KeyType": "rsa",
                "Tags": [],
                "CreateTime": "2023-11-14T11:28:30.040000+00:00"
            }
        ]
    }
    ```

3. create security group called *MySecurityGroup*
    ```
    C:\Users\ASUS\Desktop\test_aws_cli>aws ec2 create-security-group --group-name MySecurityGroup --description "security group for go test"
    {
        "GroupId": "sg-012b78fc9befd342c"
    }
    ```

4. Assign inbound rule to the previously created group
    ```
    C:\Users\ASUS\Desktop\test_aws_cli>aws ec2 authorize-security-group-ingress --group-name MySecurityGroup --protocol tcp --port 80 --cidr 0.0.0.0/0
    {
        "Return": true,
        "SecurityGroupRules": [
            {
                "SecurityGroupRuleId": "sgr-029357d2f71a02e62",
                "GroupId": "sg-012b78fc9befd342c",
                "GroupOwnerId": "260704806922",
                "IsEgress": false,
                "IpProtocol": "tcp",
                "FromPort": 80,
                "ToPort": 80,
                "CidrIpv4": "0.0.0.0/0"
            }
        ]
    }
    ```
    
5. Create the EC2 Instance
    ```
    C:\Users\ASUS\Desktop\test_aws_cli>aws ec2 run-instances --image-id ami-03caf91bb3d81b843 --instance-type t2.micro --key-name test_key1 --security-group-ids sg-012b78fc9befd342c --subnet-id subnet-0a7d96898508a9ef9
    {
        "Groups": [],
        "Instances": [
            {
                "AmiLaunchIndex": 0,
                "ImageId": "ami-03caf91bb3d81b843",
                "InstanceId": "i-03017b15c0763ad3d",
                "InstanceType": "t2.micro",
                "KeyName": "test_key1",
                "LaunchTime": "2023-11-14T12:48:20+00:00",
                "Monitoring": {
                    "State": "disabled"
                },
                "Placement": {
                    "AvailabilityZone": "ap-southeast-1c",
                    "GroupName": "",
                    "Tenancy": "default"
                },
                "PrivateDnsName": "ip-172-31-12-50.ap-southeast-1.compute.internal",
                "PrivateIpAddress": "172.31.12.50",
                "ProductCodes": [],
                "PublicDnsName": "",
                "State": {
                    "Code": 0,
                    "Name": "pending"
                },
                "StateTransitionReason": "",
                "SubnetId": "subnet-0a7d96898508a9ef9",
                "VpcId": "vpc-0b945ea58ab43d9cf",
    -- More  --
    ```
    
