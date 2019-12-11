# Ideas and instructions:
# https://github.com/burakince/docker-dotnet-sonarscanner/blob/master/Dockerfile
# https://docs.microsoft.com/en-us/dotnet/core/linux-prerequisites?tabs=netcore30
# https://sonarcloud.io/documentation/analysis/scan/sonarscanner-for-msbuild/

FROM node as nodejs

FROM jenkins/jenkins:latest
COPY --from=nodejs /usr/local/bin/node /usr/local/bin/node

RUN /usr/local/bin/install-plugins.sh git mstest matrix-auth workflow-aggregator docker-workflow blueocean credentials-binding trilead-api ssh-credentials config-file-provider nodejs

# Setup environment
ENV SONAR_SCANNER_MSBUILD_VERSION=4.7.1.2311 \
    SONAR_SCANNER_VERSION=3.2.0.1227 \
    DOTNET_SDK_VERSION=3.0 \
    SONAR_SCANNER_MSBUILD_HOME=/opt/sonar-scanner-msbuild \
    DOTNET_PROJECT_DIR=/project \
    DOTNET_SKIP_FIRST_TIME_EXPERIENCE=true \
    DOTNET_CLI_TELEMETRY_OPTOUT=true \
	JENKINS_USER="winnovator" \
	JENKINS_PASS="G#tXces2thaWInnovator15GR3@tS3cr#ts" \
	JAVA_OPTS=-Djenkins.install.runSetupWizard=false

COPY executors.groovy /usr/share/jenkins/ref/init.groovy.d/
COPY default-user.groovy /usr/share/jenkins/ref/init.groovy.d/

# DOTNET CORE

 # Switch to root to install .NET Core SDK
USER root

# Just for my sanity... Show me this distro information!
RUN uname -a && cat /etc/*release

# Based on instructiions at https://docs.microsoft.com/en-us/dotnet/core/linux-prerequisites?tabs=netcore30
# Install depency for dotnet core 3.
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
    curl libunwind8 gettext apt-transport-https && \
    curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg && \
    mv microsoft.gpg /etc/apt/trusted.gpg.d/microsoft.gpg && \
    sh -c 'echo "deb [arch=amd64] https://packages.microsoft.com/repos/microsoft-debian-stretch-prod stretch main" > /etc/apt/sources.list.d/dotnetdev.list' && \
    apt-get update

# Install the .Net Core framework, set the path, and show the version of core installed.
RUN apt-get install -y dotnet-sdk-$DOTNET_SDK_VERSION && \
    export PATH=$PATH:$HOME/dotnet && \
    dotnet --version

# Install the .Net Core framework, set the path, and show the version of core installed.
RUN apt-get install -y dotnet-sdk-2.1
	
RUN wget https://github.com/SonarSource/sonar-scanner-msbuild/releases/download/$SONAR_SCANNER_MSBUILD_VERSION/sonar-scanner-msbuild-$SONAR_SCANNER_MSBUILD_VERSION-netcoreapp2.0.zip -O /opt/sonar-scanner-msbuild.zip \
  && mkdir -p $SONAR_SCANNER_MSBUILD_HOME \
  && mkdir -p $DOTNET_PROJECT_DIR \
  && unzip /opt/sonar-scanner-msbuild.zip -d $SONAR_SCANNER_MSBUILD_HOME \
  && rm /opt/sonar-scanner-msbuild.zip \
  && chmod 775 $SONAR_SCANNER_MSBUILD_HOME/**/bin/* \
  && chmod 775 $SONAR_SCANNER_MSBUILD_HOME/**/lib/*.jar

# ENV PATH="$SONAR_SCANNER_MSBUILD_HOME:$SONAR_SCANNER_MSBUILD_HOME/sonar-scanner-$SONAR_SCANNER_VERSION/bin:${PATH}"

RUN export PATH=$PATH:$SONAR_SCANNER_MSBUILD_HOME

RUN dotnet tool install --global dotnet-sonarscanner --version 4.7.1

# Install NuGet
RUN apt-get install -y nuget

# Good idea to switch back to the jenkins user.
USER jenkins

VOLUME $DOTNET_PROJECT_DIR
WORKDIR $DOTNET_PROJECT_DIR