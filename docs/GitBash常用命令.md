## 下载地址 使用华为云镜像站
https://mirrors.huaweicloud.com/git-for-windows/

## ASP.NET Core Hosting Bundle安装检查
ls "/c/Program Files/IIS/Asp.Net Core Module/V2/"

## 查找占用 8080 端口的进程 PID

netstat -ano | grep :8080

## 关闭

$ powershell -command "Stop-Process -Id 17808 -Force"

## 生成32位随机字符串 可用于JWT 密钥
openssl rand -base64 32
## 编译docfx
docfx docfx.json --serve
## 创建.gitignore 
dotnet new gitignore
## Npgsql Database-First
dotnet ef dbcontext scaffold "Host=localhost;Port=5432;Database=MyTodoDatabase;Username=postgres;Password=123456" Npgsql.EntityFrameworkCore.PostgreSQL --output-dir Models --force