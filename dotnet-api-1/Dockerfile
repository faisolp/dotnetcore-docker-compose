ARG DOTNET_VERSION=9.0

FROM mcr.microsoft.com/dotnet/aspnet:${DOTNET_VERSION} AS runtime
WORKDIR /app
EXPOSE 80
EXPOSE 443

# คัดลอกไฟล์ publish จากเครื่องโฮสต์ (ไม่มีการ build ในไฟล์นี้)
COPY ./publish/ .

# ตั้งค่า Environment Variables
ENV ASPNETCORE_URLS=http://+:80;https://+:443

# เรียกใช้แอปพลิเคชัน (รับชื่อไฟล์จาก ARG)
ARG APP_DLL=YourApiProject.dll
ENTRYPOINT ["dotnet", "$APP_DLL"]