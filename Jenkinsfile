pipeline {
    // STEPS IN OC (Premises)
    /*
     * # Creacion de proyecto
     * oc new-project hello-openshift
     *
     * # Creacion de application
     * oc new-app redhat-openjdk18-openshift:1.5~https://github.com/lozanotux/hello-openshift
     *
     * # Creacion de Jenkins
     * oc new-app --template=jenkins-persistent
     *
     * # Creacion de pipeline
     * oc new-build --strategy=pipeline https://github.com/lozanotux/hello-openshift.git --name=hello-openshift-pipeline
     *
     * You can read more about JenkinsStrategy client here:
     * https://github.com/openshift/jenkins-client-plugin#centralizing-cluster-configuration
     */
    agent {
        label "maven"
    }
    options {
        skipDefaultCheckout()
        disableConcurrentBuilds()
    }
    stages { 
        stage('Checkiut') {
            steps {
                // scm is a objets that contains all the git information
                checkout(scm)
            }
        }
        stage('Compile') {
            steps {
                sh "mvn package -DskipTests"
            }
        }
        stage('Test') {
            steps {
                sh "mvn test"
            }
        }
        stage('Build Image') {
            /*
             * A BuildConfig a DeployConfig has Triggers, that can be dissabled.
             * Normally, we need to maintain versions into BuildConfig... so, 
             * In the customers we remove the triggers into BuildConfig.
             * Rollout can go back to previous version, but rollout is a deploy,
             * not a Rollback!.
             */
            steps {
                openshift.withCluster {
                    openshift.withProject {
                        env.TAG = readMavenPom().getVersion()
                        // Search de BuildConfig previosuly executed by oc new-app in step (2)
                        openshift.selector("bc", "hello-openshift")
                            /*
                             * At this point this startBuild search in target folder all jars
                             * to copy into image (S2I)
                             */
                            .startBuild("--from-dir=./target", "--wait=true")
                    }
                }
            }
        }
        stage('Deploy Image') {
            steps {
                script {
                    openshift.withCluster {
                        openshift.withProject("hello-openshift") {
                            openshift.set("triggers", "dc/hello-openshift", "--remove-all")
                            openshift.set("triggers", "dc/hello-openshift", "--from-image=hello-openshift:latest", "-c hello-openshift")
                            openshift.selector("dc", "hello-openshift").rollout().status()                        }
                    }
                }
            }
        }
    }
}