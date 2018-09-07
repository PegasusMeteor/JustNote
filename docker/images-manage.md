# Docker 镜像管理  

## Docker commit   
基于现有镜像制作新的镜像。  



## Docker tag   
给某个镜像打标签  

一个镜像可以多个标签。  


## 修改镜像默认运行命令

docker commit -c  'CMD ["/bin/httpd","-f","-h","/data/html"]'


## 制作镜像，并push到registry中

## 镜像的导入和导出   

docker image save 

docker image load 



