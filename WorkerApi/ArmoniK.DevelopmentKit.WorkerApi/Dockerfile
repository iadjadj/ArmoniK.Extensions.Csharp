#See https://aka.ms/containerfastmode to understand how Visual Studio uses this Dockerfile to build your images for faster debugging.

FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS base
WORKDIR /app

FROM base AS tools

RUN mkdir -p /root/.vs-debugger
WORKDIR /root/.vs-debugger

RUN apt-get update && apt-get install -y curl
RUN curl -sSL https://aka.ms/getvsdbgsh > GetVsDbg.sh
RUN bash GetVsDbg.sh -v vs2019 -r linux-x64 -l vs2019
RUN bash GetVsDbg.sh -v vs2022 -r linux-x64 -l vs2022

FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src
COPY ["WorkerApi/ArmoniK.DevelopmentKit.WorkerApi/ArmoniK.DevelopmentKit.WorkerApi.csproj", "WorkerApi/ArmoniK.DevelopmentKit.WorkerApi/"]
COPY ["Common/ArmoniK.DevelopmentKit.Common/ArmoniK.DevelopmentKit.Common.csproj", "Common/ArmoniK.DevelopmentKit.Common/"]
COPY ["Client/src/ArmoniK.DevelopmentKit.Client.Common/ArmoniK.DevelopmentKit.Client.Common.csproj", "Client/src/ArmoniK.DevelopmentKit.Client.Common/"]
COPY ["WorkerApi/ArmoniK.DevelopmentKit.WorkerApi.Common/ArmoniK.DevelopmentKit.WorkerApi.Common.csproj", "WorkerApi/ArmoniK.DevelopmentKit.WorkerApi.Common/"]
RUN dotnet restore "WorkerApi/ArmoniK.DevelopmentKit.WorkerApi/ArmoniK.DevelopmentKit.WorkerApi.csproj"
COPY . .
RUN find \( -iname obj -o -iname bin \) -exec rm -rf {} +
WORKDIR "/src/WorkerApi/ArmoniK.DevelopmentKit.WorkerApi"
RUN dotnet build "ArmoniK.DevelopmentKit.WorkerApi.csproj" -c Debug -f net6.0 -o /app/build

FROM build AS publish
RUN dotnet publish "ArmoniK.DevelopmentKit.WorkerApi.csproj" -c Deubug -f net6.0 -o /app/publish -r linux-x64 --self-contained true

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENV LD_LIBRARY_PATH=.:$LD_LIBRARY_PATH

RUN groupadd --gid 5000 armonikuser && useradd --home-dir /home/armonikuser --create-home --uid 5000 --gid 5000 --shell /bin/sh --skel /dev/null armonikuser

COPY --from=tools /root/.vs-debugger /home/armonikuser/.vs-debugger
RUN mkdir /cache && chown armonikuser: /cache && chown -R armonikuser: /home/armonikuser/.vs-debugger
USER armonikuser

ENTRYPOINT ["dotnet", "ArmoniK.DevelopmentKit.WorkerApi.dll"]
