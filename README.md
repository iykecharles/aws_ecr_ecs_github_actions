<u><strong>Deploying Go to AWS Fargate with Docker, ECR, ECS, and GitHub Actions</strong></u>

This project was all about taking a Go application I'd been working on and making it ready for the cloud. I wanted to learn how to containerize it with Docker and deploy it on AWS using ECR and ECS. It was a bit of a learning curve, but I'm really happy with the result.

**The Go Application – The Foundation**

First things first, I had to make sure the Go application itself was solid. This involved writing the actual code, of course, handling dependencies, and doing a lot of local testing. I won't go into the specifics of the Go code itself here, as the focus was on the deployment side of things. Let's just say it was a standard Go project, nothing too fancy, but it did what it was supposed to do.

Docker – Building the Container:

The next big step was creating a Docker image. This was where I really started to get into the details of containerization. I created a Dockerfile, which is like a recipe for building the image. I started with a base image – golang:1.20.5-alpine3.18 for the build stage. Alpine Linux is tiny, which makes the resulting image smaller and faster. Inside the Dockerfile, I set the working directory, copied my Go code, and then ran go mod download to grab all the necessary dependencies.   

Then came the build process: go build -o main .. This compiled my Go application into a binary called main. For security, I also created a non-root user and group within the image using addgroup and adduser. This is a best practice to avoid running the application as root inside the container. Finally, I switched to a smaller image alpine:latest for the runtime stage, copied the binary, and set the correct permissions and the user. I also exposed port 8080, which my application uses.

Building the image with docker build -t golang-app . was pretty straightforward, but I spent some time making sure everything was set up correctly within the Dockerfile. I ran the image locally with docker run to confirm that the application was running inside the container as expected. This was really helpful for catching early issues.

![dockerfile](https://github.com/user-attachments/assets/28780f32-a1db-44c3-9064-c4d664bf5003)

**GitHub Actions – Automating the Process**

To automate my Go app's deployment, I built a GitHub Actions workflow that triggers on pushes to the main branch. It first checks out the code and securely configures AWS credentials using an IAM role and stored secrets. Then, it logs into ECR, builds the Docker image, tags it with the correct ECR URI, and finally pushes it to my ECR repository, ready for deployment to ECS. This setup automates the entire build-and-push process, streamlining my workflowECR – Storing the Image in the Cloud:

With the Docker image built, I needed a place to store it in the cloud. That's where Amazon ECR comes in. I went into the AWS console and created a private ECR repository named golang-app.

Then, I had to authenticate my local Docker client with ECR using the aws ecr get-login-password command. This was a bit tricky at first, getting the AWS CLI configured correctly, but once I had that sorted, it was smooth sailing. I tagged my local Docker image with the full ECR URI – something like 827950560876.dkr.ecr.us-east-1.amazonaws.com/golang-app:latest. Finally, I pushed the image to ECR with docker push. It felt pretty cool to see my image up there in the cloud!

![workflow yml file](https://github.com/user-attachments/assets/0c4d8b15-1d38-40fb-b10f-2eee5e5cc9e2)


**ECS – Orchestrating the Deployment**

The final piece of the puzzle was deploying the application to ECS. This was the most complex part, but also the most rewarding. I started by creating an ECS cluster, which is basically a group of EC2 instances (or Fargate capacity) that run your containers. I chose Fargate, which is serverless, so I didn't have to manage any EC2 instances myself.   

Then came IAM roles. I created two roles: a task execution role and a task role. The task execution role allows ECS to pull images from ECR and manage logs. The task role gives the application running inside the container the permissions it needs (in this case, read-only access to ECR, although in other cases, it could be other AWS services).   

Next, I created an ECS task definition. This is where you define the container configuration: which image to use (the one I pushed to ECR), port mappings, resource limits, logging configurations and the IAM roles.

Finally, I created an ECS service. This service manages the desired number of running tasks. I configured the service to use the task definition I created, specified the cluster, and set the desired number of tasks to 1 (for this simple example). I also configured a load balancer (Application Load Balancer) to distribute traffic to the tasks.


**Challenges and Learnings**

There were a few bumps along the way. Getting the IAM roles and policies correct was probably the trickiest part. I had some issues with permissions at first, but with a bit of debugging and careful checking of the ARNs, I managed to get it working. I also learned a lot about Docker best practices, especially regarding multi-stage builds and non-root users.

**Conclusion**

This project was a great learning experience. I now have a much better understanding of how to containerize applications with Docker and deploy them to AWS using ECR and ECS. The automated CI/CD pipeline with GitHub Actions makes the deployment process much smoother and more efficient. I'm excited to use these skills in future projects.   


Sources and related content
