pipeline {
  agent any
  parameters {
    choice (name: 'chooseNode', choices: ['Green as web2', 'Blue as web1'], description: 'Choose which Environment to Deploy: ')
  }
  environment {
    listenerARN = 'arn:aws:elasticloadbalancing:us-east-1:959221698010:listener/app/demo-web-lb/0943f762d839c91d/85a038ff87f17eb0'
    blueARN = 'arn:aws:elasticloadbalancing:us-east-1:959221698010:targetgroup/web-1/2329de46748a4102'
    greenARN = 'arn:aws:elasticloadbalancing:us-east-1:959221698010:targetgroup/web-2/6607bbc0a02a5920'
  }
  stages {
    stage('Deployment Started') {
      parallel {
        stage('Green') {
          when {
            expression {
              params.chooseNode == 'Green as web2'
            }
          }
          stages {
            stage('Offloading Green') {
              steps {
                sh """aws elbv2 modify-listener --listener-arn ${listenerARN} --default-actions '[{"Type": "forward","Order": 1,"ForwardConfig": {"TargetGroups": [{"TargetGroupArn": "${greenARN}", "Weight": 0 },{"TargetGroupArn": "${blueARN}", "Weight": 1 }],"TargetGroupStickinessConfig": {"Enabled": true,"DurationSeconds": 1}}}]'"""
              }
            }
            stage('Deploying to Green') {
              steps {
                sh '''scp -r index.html ubuntu@52.91.234.196:/usr/share/nginx/html/
                ssh -t ubuntu@52.91.234.196 -p 22 << EOF 
                sudo service nginx restart
                '''
              }
            }
            stage('Validate and Add Green for testing') {
              steps {
                sh """
                if [ "\$(curl -o /dev/null --silent --head --write-out '%{http_code}' http://3.6.126.50/)" -eq 200 ]
                then
                    echo "** BUILD IS SUCCESSFUL **"
                    curl -I http://52.91.234.196/
                    aws elbv2 modify-listener --listener-arn ${listenerARN} --default-actions '[{"Type": "forward","Order": 1,"ForwardConfig": {"TargetGroups": [{"TargetGroupArn": "${greenARN}", "Weight": 0 },{"TargetGroupArn": "${blueARN}", "Weight": 1 }],"TargetGroupStickinessConfig": {"Enabled": true,"DurationSeconds": 1}}}]'
                else
	                echo "** BUILD IS FAILED ** Health check returned non 200 status code"
                    curl -I http://52.91.234.196/
                exit 2
                fi
                """
              }
            }
          }
        }
        stage('Blue') {
          when {
            expression {
              params.chooseNode == 'Blue as web1'
            }
          }
          stages {
            stage('Offloading Blue') {
              steps {
                sh """aws elbv2 modify-listener --listener-arn ${listenerARN} --default-actions '[{"Type": "forward","Order": 1,"ForwardConfig": {"TargetGroups": [{"TargetGroupArn": "${greenARN}", "Weight": 1 },{"TargetGroupArn": "${blueARN}", "Weight": 0 }],"TargetGroupStickinessConfig": {"Enabled": true,"DurationSeconds": 1}}}]'"""
              }
            }
            stage('Deploying to Blue') {
              steps {
                sh '''scp -r index.html ubuntu@44.197.174.101:/usr/share/nginx/html/
                ssh -t ubuntu@44.197.174.101 -p 22 << EOF 
                sudo service nginx restart
                '''
              }
            }
            stage('Validate Blue and added to TG') {
              steps {
                sh """
                if [ "\$(curl -o /dev/null --silent --head --write-out '%{http_code}' http://3.110.209.118/)" -eq 200 ]
                then
                    echo "** BUILD IS SUCCESSFUL **"
                    curl -I http://44.197.174.101/
                    aws elbv2 modify-listener --listener-arn ${listenerARN} --default-actions '[{"Type": "forward","Order": 1,"ForwardConfig": {"TargetGroups": [{"TargetGroupArn": "${greenARN}", "Weight": 1 },{"TargetGroupArn": "${blueARN}", "Weight": 1 }],"TargetGroupStickinessConfig": {"Enabled": true,"DurationSeconds": 1}}}]'
                else
	                echo "** BUILD IS FAILED ** Health check returned non 200 status code"
                    curl -I http://44.197.174.101/
                exit 2
                fi
                """
              }
            }
          }
        }
      }
    }
  }
}