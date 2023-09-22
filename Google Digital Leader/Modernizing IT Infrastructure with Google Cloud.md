# Modernizing It Infrastructure with Google Cloud

## Infrastructure Modernizing

## Understanding compute options in the cloud
- On demand self service  
-> 관리자를 거칠 필요 없이 서비스를 사용하려고 하는 사용자가 즉시 서비스 사용 가능
- Broad network access  
-> 어디에서나 잘 접속될 수 있게, 다양한 클라이언트, 사용자, 국가에서 접속 가능, 지연율 낮게
- Resource pooling  
-> 데이터 소스의 분산
- Rapid elasticity  
-> 사용량에 따라 빠르게 스케일업, 스케일다운 되어야함
- Measured service  
-> 내가 사용하는 양에 대해서 측정할 수 있어야함

### Infrastructure에서 사용할 수 있는 세 가지 컴퓨팅 옵션
- Virtual machines
- Containerization
- Serverless computing

컴퓨팅은 용량이 아니라 컴퓨터 성능을 뜻함. CPU, 메모리, 그래픽 등등  
VM은 Infrastructure을 현대화 시키는 첫번째 컴퓨팅 옵션
VM은 각각의 하드웨어적 용량이 정해져있는 반면 Container은 필요한만큼 사용됨. 따라서 더 효율적. 또한 Container는 하드웨어는 말고 운영체제만 따로 재생성 및 가상화시키는(?) 형식  

서버리스 컴퓨팅은 프로비저닝이 가능 -> 배포해뒀다가 실행(누군가가 접근하거나 사용)이 되면 돌아가는 느낌, 내가 원하는 기능에 대한 비용만 지불

## Hybrid and Multi CLoud
