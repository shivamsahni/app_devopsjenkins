pipeline{
  agent any
  
  environment{
    sonar = tool name: 'sonar_scanner_dotnet'
    username = 'shivamsahni'
    registry='shivamsahni/devops'
  }
  
  options{
  timestamps()
  timeout(activity: true, time: 1, unit: 'HOURS')
  skipDefaultCheckout()
  }
  
  stages{
    stage('git checkout'){
      steps{
        echo "git checkout"
        git branch: 'feature', url: 'https://github.com/shivamsahni/app_devopsjenkins'
      }
    }
    stage('nuget restore'){
      steps{
        echo 'nuget restore'
        bat "dotnet restore"
      }
    }
    stage('start sonaqube analysis'){
      steps{
        echo "start sonarqube"
        withSonarQubeEnv('Test_Sonar'){
          bat "${sonar}\\SonarScanner.MSBuild.exe begin /n:shivam_devops /k:shivam_devops /v:1.0"
        }
      }
    }
    stage('dotnet build'){
      steps{
        echo "run clean and build"
        bat "dotnet clean"
        bat "dotnet build"
      }
    }
    stage('Run UnitTests'){
      steps{
        echo "run unittests"
        bat "dotnet test SampleDotnetWebappTests\\SampleDotnetWebAppTests.csproj -l:trx;LogFileName: SampleDotnetWebAppTestResults.xml"
      }
    }
    stage('Stop sonarqube'){
      steps{
        echo 'Stop sonarqube'
        withSonarQubeEnv('Test_Sonar'){
          bat "${sonar}\\SonarScanner.MSBuild.exe end"
        }
      }
    }
    stage('Docker Image Creation'){
      steps{
        echo "docker image creation"
        bat "dotnet publish -c Release"
        bat "docker build -t i-shivam_devops:${BUILD_NUMBER} -t i-shivam_devops:latest --no-cache ."
      }
    }
    stage('publish to dockerhub'){
      steps{
        echo "publish to dockerhub"
        bat "docker tag i-shivam_devops:${BUILD_NUMBER} ${registry}:${BUILD_NUMBER}"
        bat "docker tag i-shivam_devops:latest ${registry}:latest"
        withDockerRegistry([credentialsId: 'DockerHub', url: ""]){
          bat "docker push ${registry}:latest"
          bat "docker push ${registry}:${BUILD_NUMBER}"
        }
      }
    }

  }
  post{
    always{
      echo 'test report generation'
      xunit([MSTest(deleteOutputFiles: true, failIfNotNew: true, pattern: "SampleDotnetWebAppTest\\TestResults\\SampleDotnetWebAppTestResults.xml, skipNoTestFile: true, stopProcessingIfError: true")])
    }
  }

}
