# EKS Cluster Setup and 2048 Game Deployment

This project is creating an Amazon EKS cluster and deploy the 2048 game application into the cluster.

## Table of Contents
- [Task 1: Create an EKS Cluster](#task-1-create-an-eks-cluster)
- [Task 2: Add Node Groups to the Cluster](#task-2-add-node-groups-to-the-cluster)
- [Task 3: Authenticate to the Cluster](#task-3-authenticate-to-the-cluster)
- [Task 4: Create a New POD for the 2048 Game](#task-4-create-a-new-pod-for-the-2048-game)
- [Task 5: Setup Load Balancer Service](#task-5-setup-load-balancer-service)
- [Task 6: Cleanup](#task-6-cleanup)

## Task 1: Create an EKS Cluster

1. **Cluster Name:** `eks-cluster`
2. **Kubernetes Version:** 1.30

![Screenshot 2024-07-10 042647](https://github.com/shivxm03/eks-2048-game-deployment/assets/157244434/994e81ef-b943-41a0-aeba-6a1e70d87d52)

### IAM Roles

- **Cluster Role:**
  - Name: `eks-cluster-role`
  - Policy: `AmazonEKSClusterPolicy`
 
![Screenshot 2024-07-10 042319](https://github.com/shivxm03/eks-2048-game-deployment/assets/157244434/37a7cb68-cb6f-4cb0-860a-cb87cd89c97c)

- **Node Group Role:**
  - Name: `node-grp-eks`
  - Policies:
    - `AmazonEKSWorkerNodePolicy`
    - `AmazonEC2ContainerRegistryReadOnly`
    - `AmazonEKS_CNI_Policy`
   
![Screenshot 2024-07-10 042336](https://github.com/shivxm03/eks-2048-game-deployment/assets/157244434/f722db91-75e1-408c-9865-2c58e9c8b3fc)

### Configuration

- **VPC:** Default VPC
- **Subnets:** Choose 2 or 3 subnets
- **Security Group:** Open ports 22, 80, 8080
- **Cluster Endpoint Access:** Public

Click 'Create'. This process takes 10-12 minutes. Wait until cluster shows up as Active.

![Screenshot 2024-07-10 042452](https://github.com/shivxm03/eks-2048-game-deployment/assets/157244434/71695b60-585c-4c47-ba0f-25fbc3e583c1)

## Task 2: Add Node Groups to the Cluster

1. **Node Group Name:** `eks-node-grp`
2. **Role:** Select the previously created role `node-grp-eks`
3. **AMI:** Default (Amazon Linux 2)
4. **Desired/Minimum/Maximum Capacity:** 1
5. **Enable SSH Access:** Choose a security group that allows ports 22, 80, 8080

Node group creation takes 2-3 minutes.

![Screenshot 2024-07-10 042513](https://github.com/shivxm03/eks-2048-game-deployment/assets/157244434/3f10b263-8988-47aa-a409-bd4f6c6f3151)

## Task 3: Authenticate to the Cluster

### Steps

1. Open AWS CloudShell.
2. Run the following command to get your account and user ID details:
   ```sh
   aws sts get-caller-identity

![Screenshot 2024-07-10 041910](https://github.com/shivxm03/eks-2048-game-deployment/assets/157244434/2580cb4f-d1ae-4f04-881f-d4e8a79f0768)

3. Create a kubeconfig file:
```sh
aws eks update-kubeconfig --region <region-code> --name <yourname>-eks-cluster
```
![Screenshot 2024-07-10 042013](https://github.com/shivxm03/eks-2048-game-deployment/assets/157244434/4ff858e7-7edf-4c87-b5df-18bd395c75c7)

4. Verify the nodes:
```sh
kubectl get nodes
```

## Task 4: Create a New POD for the 2048 Game

1. Create a YAML file for the 2048 game pod:
```sh
nano 2048-pod.yaml
```
**2048-pod.yaml**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: 2048-pod
  labels:
    app: 2048-ws
spec:
  containers:
    - name: 2048-container
      image: blackicebird/2048
      ports:
        - containerPort: 80
```
2. Apply the configuration to create the pod:
```sh
kubectl apply -f 2048-pod.yaml
```

3. View the newly created pod:
```sh
kubectl get pods
```
![Screenshot 2024-07-10 042131](https://github.com/shivxm03/eks-2048-game-deployment/assets/157244434/b691901a-1c87-42a0-a8f6-f9fe2ad412a6)

## Task 5: Setup Load Balancer Service

1. Create a YAML file for the service:
```sh
nano mygame-svc.yaml
```
**mygame-svc.yaml**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mygame-svc
spec:
  selector:
    app: 2048-ws
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
```
2. Apply the configuration:
```sh
kubectl apply -f mygame-svc.yaml
```
3. View details of the service:
```sh
kubectl describe svc mygame-svc
```
![Screenshot 2024-07-10 042202](https://github.com/shivxm03/eks-2048-game-deployment/assets/157244434/26d44780-134c-4622-9f12-40ef1f7c57a8)

4. Access the LoadBalancer
Use the LoadBalancer Ingress on the EC2 instance:
```sh
curl <LoadBalancer_Ingress>:<Port_number>
```
Example:
sh
Copy code
curl a06aa56b81f5741268daca84dca6b4f8-694631959.us-east-1.elb.amazonaws.com:80

- Go to the EC2 console, get the DNS name of the ELB, and paste it into the browser address bar to access the 2048 game.

## 2048 Game is Up and Running.

![Screenshot 2024-07-10 041841](https://github.com/shivxm03/eks-2048-game-deployment/assets/157244434/652b1991-0d20-4b22-a474-af2ad10a1267)

## Task 6: Cleanup

1. Delete the pod:
```sh
kubectl delete -f 2048-pod.yaml
```
2. Delete the service:
```sh
kubectl delete -f mygame-svc.yaml
```
## Conclusion
I have successfully created an EKS cluster, deployed the 2048 game, and accessed it via a LoadBalancer. Finally, cleaning up the resources to avoid unnecessary costs.
