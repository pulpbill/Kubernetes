This command will create the neccesary files inside a folder with your chart name:
```
helm create <your-chart>
```

If you don't have a ConfigMap inside your-chart/templates/  Let's make one:
```
vim your-chart/templates/cm.yaml
```

Now, let's add a nginx basic conf for our cm (notice how we use the "data" key):
```
apiVersion: v1
data:
  nginx.conf: |
    events {
      worker_connections  1024;
    }
    http {
      server {
        listen 80;
        location / {
          return 200 "===============================\n\n   This is your helm deploy!   \n\n===============================\n";
        }
      }
    }
kind: ConfigMap
metadata:
  name: nginx-config
```

Now, point our nginx Deployment template to the previous ConfigMap. Add the following lines to your-chart/templates/deployment.yaml:
```
volumes:
  - name: config
    configMap:
      name: nginx-config
volumeMounts:
  - name: config
    mountPath: /etc/nginx/nginx.conf
    subPath: nginx.conf
```

Run the following command to check what objects our chart will have and the values that will generate:
```
helm template <your-chart>
```

Let's install it:
```
helm install <your-chart> --name=my-app-name
```

Then, check that Service and Deploy have been created, you should be able to curl your clusterIP and get the message we set at "data" in our ConfigMap.
```
kubectl get svc
```

You can package your chart to a tar.gz file:
```
helm package <your-chart>
```

This command helps you find issues with your chart’s formatting or information:
```
helm lint <your-chart>
```

Source: https://medium.com/containerum/how-to-make-and-share-your-own-helm-package-50ae40f6c221
