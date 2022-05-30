## K8s.First Deployment

### Screenshots
![Image 1](deploy_k8s/index.png)

 
![Image 2](deploy_k8s/mnt.png)

### Main task
```bash
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webserver
  labels:
    app: simple-web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: simple-web
  template:
    metadata:
      labels:
        app: simple-web
    spec:
      containers:
      - name: web-nginx
        image: nginx
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 50m
            memory: 50Mi
          limits:
            cpu: 100m
            memory: 100Mi
        volumeMounts:
        - name: persistent-storage
          mountPath: "/usr/share/nginx/html/mnt"
        - name: new-index-mount
          mountPath: /usr/share/nginx/html/index.html
          subPath: index.html
      volumes:
      - name: persistent-storage
        persistentVolumeClaim:
          claimName: app01-pv-claim
      - name: new-index-mount
        configMap:
          name: new-index
---
apiVersion: v1
kind: Service
metadata:
  name: simple-web-service
  labels:
    run: simple-web-service
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    app: simple-web
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-sa
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/server-alias: "app.k8s-38.sa"
spec:
  rules:
    - host: app.k8s-37.sa
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: simple-web-service
                port:
                  number: 80
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: app01-pv-volume
  labels:
    type: nfs
    name: app-nfs-vol
spec:
  capacity:
    storage: 80Mi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /mnt/IT-Academy/nfs-data/sa2-20-22/alexandr_nefedin
    server: 192.168.37.105

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app01-pv-claim
  labels:
    app: simple-web
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 80Mi
  selector:
    matchLabels:
      name: app-nfs-vol
      type: nfs
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: new-index
data:
  index.html: |
    <HTML>
    <HEAD>
    <TITLE>Alexandr Nefedin</TITLE>
    <SCRIPT LANGUAGE="JavaScript">
          var sizes = new Array(0,1,2,4,8,10,12);
          sizes.pos = 0;
        
    function Elastic()
    {
        var el = document.all.elastic
        if (null == el.direction)el.direction = 1
        else if ((sizes.pos > sizes.length - 2) || (0 == sizes.pos))
        el.direction *= -1
        el.style.letterSpacing = sizes[sizes.pos += el.direction]
    setTimeout('Elastic()',100)
    }
    
    </SCRIPT>
    <BODY  bgcolor=black  onLoad=Elastic()>
    <CENTER>
    <font color="white"><h2>Hello from <font color="yellow">Alexandr Nefedin!</font><br>
    <font color="aqua"><H1 ID="elastic" ALIGN="Center">K8s OK!!!</H1>
    <br>
    
    </body>
    </HTML>
```