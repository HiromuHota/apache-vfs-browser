def GetScmProjectName() {
    return scm.getUserRemoteConfigs()[0].getUrl().tokenize('/').last().split("\\.")[0]
}

pipeline {
    agent any
    environment {
        GITHUB_TOKEN = credentials('9e494763-394a-4837-a25f-c1e9e61a7289')
        repo         = GetScmProjectName()
        version      = 'nightly'
        dist         = '8.2.0.0-342'
        username     = 'HiromuHota'
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn -DskipTests clean install'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Deliver') {
            steps {
                sh '''
                    export PATH=$PATH:$HOME/go/bin
                    github-release delete --user $username --repo $repo --tag webspoon/$version || true
                    git tag -f webspoon/$version
                    git push -f https://${GITHUB_TOKEN}@github.com/$username/$repo.git webspoon/$version
                    github-release release --user $username --repo $repo --tag webspoon/$version --name "webSpoon/$version" --description "Auto-build by Jenkins on $(date +'%F %T %Z')" --pre-release
                    cd target
                    file=`ls | grep "pentaho-vfs-browser-$dist-[0-9]*\\.jar"`
                    github-release upload --user $username --repo $repo --tag webspoon/$version --name "$file" --file $file
                    file=`ls | grep "pentaho-vfs-browser-$dist-[0-9]*-sources\\.jar"`
                    github-release upload --user $username --repo $repo --tag webspoon/$version --name "$file" --file $file
                '''
            }
        }
    }
}
