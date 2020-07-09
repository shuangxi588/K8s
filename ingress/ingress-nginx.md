# Kubernetes Ingress Nginx安装配置

## Ingress介绍

[Ingress](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#ingress-v1beta1-networking-k8s-io)公开了从群集外部到群集内[服务的](https://kubernetes.io/docs/concepts/services-networking/service/) HTTP和HTTPS路由 。流量路由由Ingress资源上定义的规则控制。

```none
    internet
        |
   [ Ingress ]
   --|-----|--
   [ Services ]
```

可以将Ingress配置为提供服务可外部访问的URL，负载平衡流量，终止SSL / TLS并提供基于名称的虚拟主机。一个[入口控制器](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers)负责履行入口，通常有一个负载均衡器，虽然它也可以配置您的边缘路由器或额外的前端，以帮助处理流量。

Ingress不会公开任意端口或协议。将HTTP和HTTPS以外的服务暴露给Internet通常使用类型为[Service.Type = NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#nodeport)或 [Service.Type = LoadBalancer的服务](https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer)。

## 先决条件

您必须具有一个[Ingress控制器](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers)才能满足Ingress的要求。仅创建Ingress资源无效。

您可能需要部署一个Ingress控制器，例如[ingress-nginx](https://kubernetes.github.io/ingress-nginx/deploy/)。您可以从许多 [Ingress控制器中进行选择](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers)。

理想情况下，所有Ingress控制器都应符合参考规范。实际上，各种Ingress控制器的操作略有不同。

## Ingress控制器的选型对比

一般行业使用最广泛的是nginx,但是nginx-ingress控制器又分社区版本和Nginx公司出的免费和收费版本。Nginx公司开发的nginx-ingress高级功能需要收费，如session保持，LDAP认证等，社区版本已经发展很多年，建议使用社区版本的ingres-nginx，本方主要介绍Kubernetes开源版本的ingress-nginx

## 官网参考资料

- 社区版本：

https://github.com/kubernetes/ingress-nginx

https://kubernetes.github.io/ingress-nginx/

https://github.com/kubernetes/ingress-nginx/blob/master/docs/user-guide/nginx-configuration/annotations.md

- Nginx公司定制的开源版本：

https://docs.nginx.com/nginx-ingress-controller/configuration/global-configuration/configmap-resource/

## 使用Helm部署ingress-nginx

1. 下载chart文件

```shell
$ helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
$ helm repo update
```

2. 定制chart上传至公司私库

```shell
$ helm fetch ingress-nginx/ingress-nginx
$ helm push ingress-nginx-<version>.tar.gz  私库名
```

3. 安装chart,helm v2

```shell
$ helm install --name ingress-nginx --namespace ingress-nginx 私库名/ingress-nginx --version=2.10.0
```

4. 如需删除chart，使用如下方法

```shell
$ helm delete --purge ingress-nginx
```

5. 查看安装服务

```shell
$ kubectl -n ingress-nginx get pods
NAME                                            READY   STATUS    RESTARTS   AGE
ingress-nginx-controller-57pkn                  1/1     Running   0          23h
ingress-nginx-controller-8mnvh                  1/1     Running   0          23h
ingress-nginx-controller-fkkvq                  1/1     Running   0          23h
ingress-nginx-controller-szhc4                  1/1     Running   0          23h
ingress-nginx-controller-tj5px                  1/1     Running   0          23h
ingress-nginx-controller-tn65r                  1/1     Running   0          23h
ingress-nginx-defaultbackend-78b4d668ff-4c665   1/1     Running   0          23h
```

## 部署应用验证Ingress服务

 前提先部署一个应用，应用部署方式略.

- 部署一个ingress服务

```shell
$ vi nginx-demo-ingress.yaml 
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
  generation: 1
  labels:
  name: nginx-demo
  namespace: dev
spec:
  rules:
  - host: nginx-demo.aegonthtf.com
    http:
      paths:
      - backend:
          serviceName: nginx-demo
          servicePort: http
        path: /
```

- 部署一个会话保持的ingress服务,支持传统应用

```shell
$ vi nginx-demo-ingress.yaml 
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/affinity: "cookie"
    nginx.ingress.kubernetes.io/session-cookie-name: "route"
    nginx.ingress.kubernetes.io/session-cookie-expires: "172800"
    nginx.ingress.kubernetes.io/session-cookie-max-age: "172800"
  creationTimestamp: null
  generation: 1
  labels:
  name: nginx-demo
  namespace: dev
spec:
  rules:
  - host: nginx-demo.aegonthtf.com
    http:
      paths:
      - backend:
          serviceName: nginx-demo
          servicePort: http
        path: /
```

- 部署一个自动跳转，支持ssl的ingress服务

```shell
1. 先创建一个公司的ssl secret
2. 创建ingress-ssl
$ vi nginx-demo-ingress-ssl.yaml 
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
  creationTimestamp: null
  generation: 1
  labels:
  name: nginx-demo
  namespace: dev
spec:
  rules:
  - host: nginx-demo.aegonthtf.com
    http:
      paths:
      - backend:
          serviceName: nginx-demo
          servicePort: http
        path: /
  tls:
  - hosts:
    - nginx-demo.aegonthtf.com
    secretName: nginx-tls

```

