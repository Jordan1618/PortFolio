## **How Will I Proceed :**

- 1) Preparing Ubuntu
- 2) Windows : Install Vector Agent + Script PS1
- 3) Test if Loki get Windows's data
- 4) Modify n8n Workflow
- 5) Global Test

## **What Am I Aiming To ?**

- 1) Collecting Various OS Event Logs
- 2) Having a System Inventory
- 3) Analyzing in real time my network traffic
- 4) Predicting risks and make recommandations

## **1) Looking for Ubuntu ?**

- cd /root/vector && docker compose up -d
- docker ps | grep vector
- ss -tlnp | grep 9000
- curl -s -o /dev/null -w "HTTP %{http_code}\n" -X POST http://localhost:9000 -H "Content-Type: application/json" -d '{"test":"ping","level":"info","hostname":"test","source_os":"linux","site":"default","log_type":"test","message":"test"}'