# 26 — Containers

## 1. מה זה?
ECS, Fargate ו-EKS מריצים containers ב-AWS.
## 2. למה צריך את זה?
Containers מתאימים לשירותים ארוזים, workloads ארוכים ו-portability; הבחירה קובעת את רמת הניהול.
## 3. איך זה עובד?
ECS הוא orchestrator AWS-native. Fargate הוא serverless compute ל-ECS/EKS—אין EC2 nodes לניהול. EKS מספק Kubernetes מנוהל אך עדיין דורש הבנת Kubernetes/nodes או Fargate. ECR מאחסן images.
## 4. הדברים שחייבים לדעת למבחן
- least operational overhead containers → ECS on Fargate.
- Kubernetes requirement → EKS.
- ECS EC2 נותן שליטה/חיסכון אפשרי אך מנהל hosts.
- task role נותן permissions ל-container.
## 5. ההבדלים החשובים
| בחירה | מתי |
|---|---|
| ECS Fargate | managed containers פשוט |
| ECS EC2 | שליטה על hosts/capacity |
| EKS | Kubernetes ecosystem |
## 6. מלכודות במבחן
Fargate אינו orchestrator; הוא launch type/compute. EKS אינו בהכרח least operational overhead.
## 7. Scenario מהעולם האמיתי
API containerized ללא צורך ב-Kubernetes: ECS service ב-Fargate מאחורי ALB, images ב-ECR ו-task role מינימלי.
## 8. מה לא צריך לדעת
לא לשנן Kubernetes manifests.
## 9. סיכום
- ECS=orchestration AWS.
- Fargate=no servers.
- EKS=Kubernetes.
- ECR=image registry.
- role לכל task.
## 10. בדיקת הבנה
1. מה הכי פשוט ל-containers?
2. מה בוחרים ל-Kubernetes?
3. מה נותן permissions ל-task?
+

## 5. עלות, תמחור ו-trade-offs
ECS על EC2 מחויב instance-hours ו-EBS אך יכול להיות זול ויעיל לעומס יציב; Spot מוזיל workloads הניתנים להפרעה. Fargate מחויב לפי vCPU וזיכרון בזמן task, יקר יותר ל-idle אך ללא ניהול hosts. EKS מוסיף cluster charge לצד nodes/Fargate; בוחרים בו כש-Kubernetes ecosystem הוא דרישה, לא כברירת מחדל.

## 6. Well-Architected view
- **Operational Excellence:** ECS/EKS deployment strategy, health checks, logs ו-auto scaling.
- **Security:** task/pod roles, image scanning, private subnets ו-least privilege.
- **Reliability:** multi-AZ services, desired count, ALB health checks ו-rolling deployments.
- **Performance Efficiency:** requests/limits, right-size tasks, placement ו-load testing.
- **Cost Optimization:** Fargate ל-bursty, EC2/Spot ליציב; מחיקת unused tasks/images.
- **Sustainability:** bin packing ו-scale-in מצמצמים idle nodes.

## 7. מלכודות ו-Scenario
Fargate הוא compute ולא orchestrator; EKS עדיין דורש Kubernetes operations. API ללא צורך ב-Kubernetes: ECS Fargate מאחורי ALB, ECR image ו-task role.
