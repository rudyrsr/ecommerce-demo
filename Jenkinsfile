#!groovy

env.MIN_VERSION="v1.";

pipeline {

	//agent none
	
	agent {
	    node {
		label 'master'
	    }
	}

	options {
    		timestamps()
  	}

	environment {
	    MyKeyID="myCustomValue1"
		folderTrabajo="MiCarpeta-${BUILD_ID}"
	}
	
	stages {

		stage('primera etapa') {
        		steps {
        			script {
              				node {
						timestamps  {
							println "primera etapa"
							if(isUnix()) {
								echo "El nodo actual es un nodo linux"

								def secret;
								withCredentials([string(credentialsId: "AccessTokenPrueba", variable: "AccessToken")]) {
									withEnv( ["JAVA_HOME=JavaPath"]  ) {
										println "AccessToken: $AccessToken";
										secret = "$AccessToken"
									}
								}

								echo "AccessToken2: $secret";
								sh "env"

								checkout scm

								env.MIN_VERSION=sh(returnStdout: true, script: "git rev-parse --short HEAD").trim()
								println "La version actual es: ${MIN_VERSION}"
							}
							 else {
								 println "El nodo actual es un nodo windows"
							 }
						}	
              			}
            		}
		}
		post {
			success {
				echo 'La primera etapa se ejecuto existosamente'
			}
			failure {
				echo 'La ejecucion de la primera etapa del pipeline ha fallado :('
			}
		}
    	} // Cerrar bloque primera etapa
		
	stage('Ejecutar unit tests en paralelo') {
            parallel {
                stage('Test On Windows') {
                    agent {
                        label "master"
                    }
                    steps {
                        println "Ejecutando los tests de windows"
                    }
                }
                stage('Test On Linux') {
                    agent {
                        label "master"
                    }
                    steps {
                        println "Ejecutando los tests en linux"
                    }
                }
            }
        }
	
    	stage('Init') {
    	//agent {
                    //docker { image 'node:7-alpine' }
                //}
        	steps {
        		script {
              		node {
    				docker.withRegistry('https://registry.hub.docker.com/',"DockerHubCredentials") {
    					docker.image('98640321id/nodejs:pipeline').inside("-u root:root") {
							timestamps  {
    						  println "Descargar codigo fuente"
							  	dir("$folderTrabajo") {
	    						
	    							  checkout scm
								  //MIN_VERSION=sh(returnStdout: true, script: "git rev-parse --short HEAD").trim()
	    							  sh """
								  	git --version
	    							  	npm --version
	    								npm install
	    								"""
    							     }
								}
    						
    							
    						
    				  		}
						    stash name: "${folderTrabajo}", include: "${folderTrabajo}/**"
    					}
            			
              		}
            	}
        	}
    	} // Cerrar bloque Init
    		
    		
    	stage('Analisis de codigo con Sonar') {
    		steps {
    			script {
    				node {
    					docker.withRegistry('https://registry.hub.docker.com/',"DockerHubCredentials") {
    						docker.image('98640321id/sonar_cli:scanner').inside("-u root:root") {
						//docker.image('98640321id/nodejs:pipeline').inside("-u root:root") {
							//ws {
    						      timestamps  {
							      //unstash "${folderTrabajo}"
							      
							      dir("${folderTrabajo}") {
								      checkout scm
    								 sh """
    								 	echo "Analisis de codigo con Sonar"
									/tmp/sonar-scanner-3.0.2.768/bin/sonar-scanner -D"sonar.version=${MIN_VERSION}"
    								    """	
    								}
    						      		
							}
    						}
    					}

    				}
    			}
    		}
    	} // "Cerrar Analisis de codigo con Sonar"
    		
    	stage('Contenedor Docker') {
    		steps {
    			script {
    				node {
    					//docker.withRegistry('https://registry.hub.docker.com/',"DockerHubCredential2") {
    						//docker.image('98640321id/nodejs:pipeline').inside("-u root:root") {
    						      timestamps  {
    							  unstash "${folderTrabajo}"
    								dir("${folderTrabajo}") {
    								 sh """
    									 docker login
    									 docker build -t ecommerce:first .
    									 docker tag  ecommerce:first 98640321id/ecommerce:first
    									 docker push 98640321id/ecommerce:first
    								    """	
    								}
    						      }
    						//}
    					//}

    				}
    			}
    		}
    	} // Cerrar bloque Contenedor Docker
    	
    	// stage('Deploy') {
        // 	steps {
        // 		script {
        //       		node {
    	// 			docker.withRegistry('https://registry.hub.docker.com/',"DockerHubCredential2") {
    	// 				docker.image('98640321id/nodejs:pipeline').inside("-u root:root") {
    	// 					unstash "${stashName}"
    	// 					dir("myFolder") {
    	// 					 sh """
    	// 						npm start 
    	// 					    """	
    	// 					}
    	// 				}
    	// 			}
        //       		}
        //     	}
        // 	}
    	// }// cerrar bloque deploy

} // Cerrar bloque stages
	
post { 
	 	always { 

	 		script {

	 			node {
	 				echo "Ejecutando function always post."
					//cleanWs()
	 				}
	 			}
        }

        success {
            echo 'El pipeline se ejecuto existosamente'
        }
        unstable {
            echo 'El estado de la ejecucion del pipeline es inestable'
        }
        failure {
            echo 'La ejecucion del pipeline ha fallado :('
        }
        changed {
            echo 'Le ejecucion del pipeline ha cambiado'
        }
		
		
}

}
