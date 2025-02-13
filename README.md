# Common Production Issues and Solutions

## 1. Website Down (Container Crashed)
### Cause / How to Create:
- Stop the container manually:
  ```bash
  docker stop smile-container
  ```
- Kill the container to simulate a crash:
  ```bash
  docker kill smile-container
  ```

### Symptoms:
- Website not accessible (HTTP error, connection refused, etc.)
- Container not running

### Troubleshooting Steps:
1. Check if the container is running:
   ```bash
   docker ps -a
   ```
2. Inspect container logs:
   ```bash
   docker logs smile-container
   ```
3. Inspect the container for errors:
   ```bash
   docker inspect smile-container
   ```

### Quick Fix:
- Restart the container:
  ```bash
  docker start smile-container
  ```

### Long-Term Solution:
- Implement restart policy in the container:
  ```bash
  docker run -d --restart=always -p 80:80 --name smile-container smile-website
  ```

---

## 2. Port Blocked / Address Already in Use
### Cause / How to Create:
- Start Nginx or another service using port 80:
  ```bash
  sudo systemctl start nginx
  ```
- Or, run another container using port 80:
  ```bash
  docker run -d -p 80:80 nginx
  ```

### Symptoms:
- Website down; Error: bind: address already in use
- Container fails to start

### Troubleshooting Steps:
1. Check which service is using port 80:
   ```bash
   sudo netstat -tulnp | grep :80
   ```
2. Identify the process ID (PID) occupying port 80:
   ```bash
   sudo lsof -i :80
   ```

### Quick Fix:
- Stop the conflicting service:
  ```bash
  sudo systemctl stop nginx
  ```
- Kill the PID:
  ```bash
  sudo kill <PID>
  ```

### Long-Term Solution:
- Configure Nginx properly or run containers on a different port:
  ```bash
  docker run -d -p 8080:80 --name smile-container smile-website
  ```

---

## 3. High CPU / Memory Usage
### Cause / How to Create:
- Simulate CPU load inside a container:
  ```bash
  docker exec -it smile-container sh
  yes > /dev/null &
  ```

### Symptoms:
- Website slow, unresponsive
- Instance lagging, SSH slow

### Troubleshooting Steps:
1. Monitor CPU and Memory Usage:
   ```bash
   docker stats
   ```
2. Identify the top processes:
   ```bash
   top
   ```
3. Find resource-hungry processes inside the container:
   ```bash
   docker exec -it smile-container top
   ```

### Quick Fix:
- Restart the container:
  ```bash
  docker restart smile-container
  ```

### Long-Term Solution:
- Limit CPU & Memory usage for the container:
  ```bash
  docker run -d --memory=512m --cpus=0.5 -p 80:80 --name smile-container smile-website
  ```

---

## 4. Disk Space Full
### Cause / How to Create:
- Create large dummy files:
  ```bash
  dd if=/dev/zero of=/tmp/dummyfile bs=1M count=5000
  ```

### Symptoms:
- Website down, container failing to start
- SSH errors like no space left on device

### Troubleshooting Steps:
1. Check disk space:
   ```bash
   df -h
   ```
2. Check Docker disk usage:
   ```bash
   docker system df
   ```

### Quick Fix:
- Clean up unused containers, images, volumes:
  ```bash
  docker system prune -a
  ```

### Long-Term Solution:
- Monitor disk usage regularly and set up alerts.
- Allocate more storage if required.

---

## 5. Container Image Not Found
### Cause / How to Create:
- Remove the image manually:
  ```bash
  docker rmi smile-website
  ```

### Symptoms:
- Container fails to start
- Error: Image not found

### Troubleshooting Steps:
1. Check if the image exists:
   ```bash
   docker images
   ```

### Quick Fix:
- Rebuild the image:
  ```bash
  docker build -t smile-website /var/www/smile/
  ```

### Long-Term Solution:
- Automate image rebuilds in CI/CD pipelines.
- Push stable images to Docker Hub and pull from there:
  ```bash
  docker push <dockerhub-username>/smile-website
  ```

---

## 6. Network Issues
### Cause / How to Create:
- Block internet on the EC2 instance via security groups or firewall rules.

### Symptoms:
- Container can’t pull images or connect to the internet.
- Website may fail if dependent on external services.

### Troubleshooting Steps:
1. Check internet from the instance:
   ```bash
   ping google.com
   ```
2. Check container connectivity:
   ```bash
   docker exec -it smile-container ping google.com
   ```

### Quick Fix:
- Restart network services:
  ```bash
  sudo systemctl restart networking
  ```

### Long-Term Solution:
- Configure proper Security Groups and Firewall Rules on AWS.

---

## 7. File Permission Issues
### Cause / How to Create:
- Change permissions on website files:
  ```bash
  chmod 000 /var/www/smile/index.html
  ```

### Symptoms:
- Website shows 403 Forbidden or 500 Internal Server Error

### Troubleshooting Steps:
1. Check permissions:
   ```bash
   ls -l /var/www/smile/
   ```

### Quick Fix:
- Fix permissions:
  ```bash
  chmod 755 /var/www/smile/
  ```

### Long-Term Solution:
- Automate permission setup in your deployment scripts (Ansible):
  ```yaml
  - name: Set permissions for Smile Website
    file:
      path: /var/www/smile/
      state: directory
      mode: '0755'
  ```

