## [visual studio Git使用](https://learn.microsoft.com/zh-cn/visualstudio/version-control/git-manage-repository?view=visualstudio)

## 下载地址 使用华为云镜像站

https://mirrors.huaweicloud.com/git-for-windows/

## ASP.NET Core Hosting Bundle 安装检查

ls "/c/Program Files/IIS/Asp.Net Core Module/V2/"

## 查找占用 8080 端口的进程 PID

netstat -ano | grep :8080

## 关闭

$ powershell -command "Stop-Process -Id 17808 -Force"

## 生成 32 位随机字符串 可用于 JWT 密钥

openssl rand -base64 32

## 编译 docfx

docfx docfx.json --serve

## 创建.gitignore

dotnet new gitignore

## Npgsql Database-First

dotnet ef dbcontext scaffold "Host=localhost;Port=5432;Database=MyTodoDatabase;Username=postgres;Password=123456" Npgsql.EntityFrameworkCore.PostgreSQL --output-dir Models --force

## SqlServer Database-First

dotnet ef dbcontext scaffold "Server=localhost;Database=test;Trusted_Connection=True;TrustServerCertificate=True;" `Microsoft.EntityFrameworkCore.SqlServer`
--table Blog `--output-dir Models`
--context-dir Data `--context TestDbContext`
--force `--no-onconfiguring`
--data-annotations `
--use-database-names

## 修改 Git 远程仓库地址为 SSH

$ git remote -v
origin https://github.com/ZQC1105/Web.git (fetch)
origin https://github.com/ZQC1105/Web.git (push)

CZQ1105@DESKTOP-0JUFRUE MINGW64 /d/Web (main)
$ git remote set-url origin git@github.com:ZQC1105/Web.git

CZQ1105@DESKTOP-0JUFRUE MINGW64 /d/Web (main)
$ git remote -v
origin git@github.com:ZQC1105/Web.git (fetch)
origin git@github.com:ZQC1105/Web.git (push)

## Vue3 git 运行命令 pnpm dev
