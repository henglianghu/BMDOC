文档中心

mike 打包
将文档推送到远程分支
mike deploy V1.0.0 latest -p

mike delete --all 清理旧版本的文档

随着你拥有的多个文档版本，可能希望设置一个默认版本，使得访问网站根目录的人会被重定向到最新的文档版本：
mike set-default [标识符]
