# [5] **Hosting a Web App on Google Cloud Using Compute Engine**


**여기서는 "Task 4. Create Compute Engine instances" 까지만 실습 수행한다 (Instance Group과 LB/CDN은 숙제)**

백엔드와 프론트엔드

[](https://www.grabbing.me/69a68655ae9c46efaeae5014b9f9034d)

### region 설정
    1. Set the project region for this lab:
        
        gcloud config set compute/region us-central1
        
    2. Create a variable for region:
        
        export REGION=us-central1
        
    3. Create a variable for zone:
        
        export ZONE=us-central1-c
        
    
## 작업 1. Enable Compute Engin API
다음을 실행하여 Compute Engine API를 사용 설정  
~~~jsx
gcloud services enable [compute.googleapis.com](http://compute.googleapis.com/)
~~~
    
## 작업 2. Create Cloud Storage bucket
Cloud Shell 내에서 다음을 실행하여 새 Cloud Storage 버킷을 만듭니다.
    
```jsx
gsutil mb gs://fancy-store-$DEVSHELL_PROJECT_ID
```
    
## 작업 3. Clone source repository
- 깃에서 코드 복제
    
    ```jsx
    git clone https://github.com/googlecodelabs/monolith-to-microservices.git
    ```
    
- ~/monolith-to-microservices에 접근
    
    ```jsx
    cd ~/monolith-to-microservices
    ```
    
- 애플리케이션이 로컬에서 실행될 수 있도록 코드의 초기 빌드를 실행
    
    ```jsx
    ./setup.sh
    ```
    
- Next, run the following to test the application, switch to the `microservices` directory, and start the web server:
    
    ```jsx
    cd microservices
    npm start
    ```
    
- 결과창
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/86231768-7c3d-4bc0-add8-cc35a895498e/Untitled.png)
    
- Preview your application by clicking the **web preview icon** then selecting **Preview on port 8080**.
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f0674973-31d7-4eac-9f0d-a7d4b1a0a34c/Untitled.png)
    
## 작업 4. Create Compute Engine instances
일부 Compute Engine 인스턴스 배포를 시작
    
1. Create a startup script to configure instances.
2. Clone source code and upload to Cloud Storage.
3. Deploy a Compute Engine instance to host the backend microservices.
4. Reconfigure the frontend code to utilize the backend microservices instance.
5. Deploy a Compute Engine instance to host the frontend microservice.
6. Configure the network to allow communication.
    
### Create the startup script
1. Open Editor 클릭해서 monolith-to-microservices 폴더로 이동
2. 폴더에서 파일>새 파일을 만들어서 이름을 startup-script.sh로 설정
3. 그 파일에 해당 코드 복붙
- 코드
            
            ```jsx
            #!/bin/bash
            # Install logging monitor. The monitor will automatically pick up logs sent to
            # syslog.
            curl -s "https://storage.googleapis.com/signals-agents/logging/google-fluentd-install.sh" | bash
            service google-fluentd restart &
            # Install dependencies from apt
            apt-get update
            apt-get install -yq ca-certificates git build-essential supervisor psmisc
            # Install nodejs
            mkdir /opt/nodejs
            curl https://nodejs.org/dist/v16.14.0/node-v16.14.0-linux-x64.tar.gz | tar xvzf - -C /opt/nodejs --strip-components=1
            ln -s /opt/nodejs/bin/node /usr/bin/node
            ln -s /opt/nodejs/bin/npm /usr/bin/npm
            # Get the application source code from the Google Cloud Storage bucket.
            mkdir /fancy-store
            gsutil -m cp -r gs://fancy-store-[DEVSHELL_PROJECT_ID]/monolith-to-microservices/microservices/* /fancy-store/
            # Install app dependencies.
            cd /fancy-store/
            npm install
            # Create a nodeapp user. The application will run as this user.
            useradd -m -d /home/nodeapp nodeapp
            chown -R nodeapp:nodeapp /opt/app
            # Configure supervisor to run the node app.
            cat >/etc/supervisor/conf.d/node-app.conf << EOF
            [program:nodeapp]
            directory=/fancy-store
            command=npm start
            autostart=true
            autorestart=true
            user=nodeapp
            environment=HOME="/home/nodeapp",USER="nodeapp",NODE_ENV="production"
            stdout_logfile=syslog
            stderr_logfile=syslog
            EOF
            supervisorctl reread
            supervisorctl update
            ```
            
1. [DEVSHELL_PROJECT_ID] 코드에서 이 부분을 찾아 나의 프로젝트 ID로 변경
2. Cloud Shell 터미널로 돌아와서 startup-script.sh파일을 버킷에 복사하는 코드 실행
        
    ```jsx
        gsutil cp ~/monolith-to-microservices/startup-script.sh gs://fancy-store-$DEVSHELL_PROJECT_ID
    ```
        
    
### Copy code into the Cloud Storage bucket
1. Copy the cloned code into your bucket:
        
    ```jsx
        cd ~
        rm -rf monolith-to-microservices/*/node_modules
        gsutil -m cp -r monolith-to-microservices gs://fancy-store-$DEVSHELL_PROJECT_ID/
    ```
        
### Deploy the backend instance
배포할 첫 번째 인스턴스는 주문 및 제품 마이크로서비스를 수용할 백엔드 인스턴스
        
1. 다음 명령을 실행하여 `e2-medium`시작 스크립트를 사용하도록 구성된 인스턴스를 생성합니다. `backend`나중에 특정 방화벽 규칙을 적용할 수 있도록 인스턴스 로 태그가 지정됨
        
    ```jsx
    gcloud compute instances create backend \
        --zone=$ZONE \
        --machine-type=e2-medium \
        --tags=backend \
        --metadata=startup-script-url=https://storage.googleapis.com/fancy-store-$DEVSHELL_PROJECT_ID/startup-script.sh
    ```
        
### Configure a connection to the backend**
1. 다음 명령을 사용하여 백엔드의 외부 IP 주소를 검색하고 `EXTERNAL_IP`백엔드 인스턴스에 대한 탭 아래를 살펴보십시오.
        
    ```jsx
    gcloud compute instances list
    ```
        
1. Example output
        
![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e296eb67-135c-4aff-a56c-69b895c260d3/Untitled.png)
        
1. **Copy the External IP** for the backend.
2. In the Cloud Shell Explorer, navigate to `monolith-to-microservices` > `react-app`.
3. In the Code Editor, select **View** > **Toggle Hidden Files** in order to see the `.env` file.
4. In the `.env` file, replace `localhost` with your `[BACKEND_ADDRESS]`:
        
```jsx
    REACT_APP_ORDERS_URL=http://[BACKEND_ADDRESS]:8081/api/orders
    REACT_APP_PRODUCTS_URL=http://[BACKEND_ADDRESS]:8082/api/products
```
        
    
https://cloud.google.com/sdk sdk설치