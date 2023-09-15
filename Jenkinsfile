pipeline {
    agent any
    tools {
        ant '1.10.14'
    }

    stages {
        stage('Create Artifactory Directory') {
            steps {
                script {
                    // Create a directory with today's date in Artifactory
                    def today = new Date().format("yyyy-MM-dd")
                    def codeDirectoryName = "code/${today}" // Modify the directory structure as needed
                    def serverId = 'Your_Artifactory_Server_Id' // Replace with your configured Artifactory server ID
                    def createDirectorySpec = """{
                        "files": [
                            {
                                "target": "${codeDirectoryName}/",
                                "flat": "true",
                                "pattern": ""
                            }
                        ]
                    }"""
                    def buildInfo = server.upload(createDirectorySpec, serverId)
                }
            }
        }

        stage('Checkout') {
            steps {
                // Checkout the code from the GitHub repository
                checkout scm
            }
        }

        stage('Code Execution') {
            steps {
                script {
                    def codeDir = '.'
                    def javaFiles = getChangedFiles(codeDir, '**/*.java')
                    def cppFiles = getChangedFiles(codeDir, '**/*.cpp')
                    def pythonFiles = getChangedFiles(codeDir, '**/*.py')

                    if (javaFiles) {
                        echo "Java code changes detected in the following files:"
                        echo javaFiles.join('\n')

                        for (def file : javaFiles) {
                            def className = file.replaceAll('.*/(.*).java', '$1')
                            def buildpath = 'Java/build.xml'
                            try {
                                bat "ant -f $buildpath -Djava.file=$file -Dmain.class=$className compile"
                                echo "Java code in $file compiled successfully."
                                // Store .class files in the Artifactory directory
                                def serverId = 'Your_Artifactory_Server_Id' // Replace with your configured Artifactory server ID
                                def targetDirectory = "${codeDirectoryName}/${className}.class" // Modify the directory structure as needed
                                def uploadSpec = """{
                                    "files": [
                                        {
                                            "pattern": "${className}.class",
                                            "target": "${targetDirectory}"
                                        }
                                    ]
                                }"""
                                def buildInfo = server.upload(uploadSpec, serverId)
                            } catch (Exception e) {
                                error "Error in $file: $e.getMessage()"
                            }
                        }
                    } else {
                        echo "No Java code changes detected in known code directories."
                    }

                    if (cppFiles) {
                        echo "C++ code changes detected in the following files:"
                        echo cppFiles.join('\n')

                        for (def file : cppFiles) {
                            // Execute C++ code here (similar to the previous stage)
                            // ...
                            // Store .exe files in the Artifactory directory
                            def serverId = 'Your_Artifactory_Server_Id' // Replace with your configured Artifactory server ID
                            def targetDirectory = "${codeDirectoryName}/${file.replaceAll('.*/(.*).cpp', '$1.exe')}" // Modify the directory structure as needed
                            def uploadSpec = """{
                                "files": [
                                    {
                                        "pattern": "${file.replaceAll('.*/(.*).cpp', '$1.exe')}",
                                        "target": "${targetDirectory}"
                                    }
                                ]
                            }"""
                            def buildInfo = server.upload(uploadSpec, serverId)
                        }
                    } else {
                        echo "No C++ code changes detected in known code directories."
                    }

                    if (pythonFiles) {
                        echo "Python code changes detected in the following files:"
                        echo pythonFiles.join('\n')

                        for (def file : pythonFiles) {
                            def result = bat(script: "python $file", returnStatus: true)
                            if (result == 0) {
                                echo "Success: Python script executed successfully."
                                // Store .py files in the Artifactory directory
                                def serverId = 'Your_Artifactory_Server_Id' // Replace with your configured Artifactory server ID
                                def targetDirectory = "${codeDirectoryName}/${file}" // Modify the directory structure as needed
                                def uploadSpec = """{
                                    "files": [
                                        {
                                            "pattern": "$file",
                                            "target": "${targetDirectory}"
                                        }
                                    ]
                                }"""
                                def buildInfo = server.upload(uploadSpec, serverId)
                            } else {
                                error "Failure: Python script execution failed with exit code $result."
                            }
                        }
                    } else {
                        echo "No Python code changes detected in known code directories."
                    }
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '**/report.txt', allowEmptyArchive: true
            emailext (
                subject: 'Jenkins Job Report',
                body: 'Please find the attached report file.',
                to: 'your_email@example.com', // Replace with the recipient email
                attachmentsPattern: '**/report.txt',
                attachLog: true
            )
        }
    }
}

@NonCPS
List<String> getChangedFiles(String directory, String filePattern) {
    def changedFiles = []
    for (changeLogSet in currentBuild.changeSets) {
        for (entry in changeLogSet.getItems()) {
            if (entry.affectedPaths.any { it.startsWith(directory) && it.endsWith(filePattern) }) {
                changedFiles.add(entry.filePath)
            }
        }
    }
    return changedFiles
}
