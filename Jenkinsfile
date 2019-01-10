properties([
    parameters([
        string (name: 'branchName', defaultValue: 'master', description: 'Branch to get the tests from')
    ])
])

def isFailed = false
def branch = params.branchName
def buildArtifactsFolder = "C:/BuildPackagesFromPipeline/$BUILD_ID"
currentBuild.description = "Branch: $branch"

node('Slave1')
{
    stage('Checkout')
    {
        git branch: branch, url: 'https://github.com/PixelScrounger/atata-phptravels-uitests.git'
    }

    stage('Restore NuGet')
    {
        powershell '.\\build.ps1 RestorePackages'
    }

    stage('Build Solution')
    {
        powershell '.\\build.ps1 Build'
    }

    stage('Copy Build Artifacts')
    {
        powershell ".\\build.ps1 CopyArtifacts -BuildArtifactsFolder $buildArtifactsFolder"
    }
}

catchError
{
    isFailed = true
    stage('Tests')
    {
        parallel FirstTest: {
            node('master')
            {
               bat "C:/Dev/NUnit.Console-3.9.0/nunit3-console.exe $buildArtifactsFolder/PhpTravels.UITests.dll --where cat==FirstTest"
            }
        }, SecondTest: {
            node('Slave1')
            {
                bat "C:/Dev/NUnit.Console-3.9.0/nunit3-console.exe $buildArtifactsFolder/PhpTravels.UITests.dll --where cat==SecondTest"
            }
        }
    }
    isFailed = false
}

node
{
    stage('Reporting')
    {
        archiveArtifacts 'TestResult.xml'
        nunit testResultsPattern: 'TestResult.xml'

        /*
        if (!isFailed)
        {
            slackSend color: "good", message: "All tests passed.\nBranch: $branch\bBuild number: $env.BUILD_NUMBER"
        }
        else
        {
            slackSend color: "danger", message: "Tests failed.\nBranch: $branch\bBuild number: $env.BUILD_NUMBER"
        }*/
    }    
}