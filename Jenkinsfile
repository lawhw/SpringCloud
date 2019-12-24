pipeline {

    options { disableConcurrentBuilds() }

    environment {
		GIT_TAG = sh(returnStdout: true,script: 'git describe --tags --always').trim()
        PROJECT_NAME = "escloud" 
		ENV_NAME = "${env.BRANCH_NAME == 'master' ? 'pro' : 'dev'}"
        TAG_NAME = "${env.BRANCH_NAME == 'master' ? 'stable' : 'test'}.${GIT_TAG}"
        //TAG_NAME = 'latest'
		REGISTRY_CREDENTIALS_ID = 'jenkins-registry-admin'
		DOCKER_ADDR = 'http://registry.apps.es1688.k8s.local'
		def MAVEN_HOME  = tool 'mvn3.6'
		def NODE_HOME = tool 'nodejs13'

		//env.PATH = "${MAVEN_HOME}/bin:${env.PATH}"
    }
	//在master节点配置jenkins的k8s云参数
    agent { label "master" }

    stages {
		//复制jenkins配置文件
        stage('准备配置文件') {
            steps {
                sh "ls -lrt && pwd"

				sh "cp -rf ${JENKINS_HOME}/mvnconf/* ${MAVEN_HOME}/conf/"
				
                //sh "mkdir -p ${JENKINS_HOME}/jenkins_config/"
                //sh "cp -rf * ${JENKINS_HOME}/jenkins_config/"
            }
        }
		//maven编译项目并打包
        stage('服务端编译') {
			steps {
				dir('./common') {
					sh "${MAVEN_HOME}/bin/mvn  install -Dmaven.test.skip=true"
				}
			   sh "${MAVEN_HOME}/bin/mvn install -Dmaven.test.skip=true"
			}
            //steps { sh '${JENKINS_HOME}/consul-template -vault-addr "http://consul.es-jr.cn" -config "jenkins_config.hcl" -once -vault-retry-attempts=1 -vault-renew-token=false' }
        }

		//在master节点上执行脚本设置jenkins配置k8s云参数，主要是jenkins-slave参数
        stage('设置云参数') {
            steps {
				  sh "ls -lrt && pwd"
                //load("/var/jenkins_home/jenkins_config/src/kubernetes.groovy")
            }
        }
		//通过dockerfile创建Docker文件
        stage('构建服务端镜像') {
            steps {
                script {
					 def prolist = ['authentication-server:auth/authentication-server/','authorization-server:auth/authorization-server/','bus-server:center/bus/','producer:demos/producer/','producer-jpa:demos/producer-jpa','gateway-admin:gateway/gateway-admin/','gateway-web:gateway/gateway-web/','admin:monitor/admin/','organization:sysadmin/organization/']
			
					for (int i = 0;i < prolist.size();++i){
						def prolistitem = prolist[i].tokenize(':')
						def proj_path = prolistitem[1]
						def proj_name = prolistitem[0]
						echo "docker build the ${prolist[i]} project"
						echo "项目名称 ${proj_name} ,路径:${proj_path} "
						docker.withRegistry('http://registry.apps.es1688.k8s.local', "${REGISTRY_CREDENTIALS_ID}") {
							docker.build("${PROJECT_NAME}-${ENV_NAME}/${proj_name}:${TAG_NAME}", "-f ${proj_path}/src/main/docker/Dockerfile ${proj_path}/target").push()
						}						
					}
                }
            }
        }
		//通过dockerfile创建Docker文件
        stage('构建管理端镜像') {
            steps {
                script {
					 //def prolist = ['authentication-server:auth/authentication-server/','authorization-server:auth/authentication-server/','bus-server:center/bus/','producer:demos/producer/','producer-jpa:demos/producer-jpa','gateway-admin:gateway/gateway-admin/','gateway-web:gateway/gateway-web/','admin:monitor/admin/','organization:sysadmin/organization/']
					 def proj_name = 'escloud-admin'
					 def proj_path = 'escloud-admin'
						//docker.withRegistry('http://registry.apps.es1688.k8s.local', "${REGISTRY_CREDENTIALS_ID}") {
						//	docker.build("${PROJECT_NAME}-${ENV_NAME}/${proj_name}:${TAG_NAME}", "-f ${proj_path}/Dockerfile ${proj_path}").push()
						//}						

                }
            }
        }

    }
}
