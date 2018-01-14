node {
  docker.image('bbrfkr0129/build_test:with_boto').inside("-v /var/jenkins_home/") {
    stage("clone git repo") {
      sh 'rm -rf iac-demo-cd && git clone https://github.com/bbrfkr/iac-demo-cd'
    }
    state("create instance for unit test") {
      sh 'cd iac-demo-cd && ansible-playbook -i hosts playbooks/create-test-instance.yaml'
      sh 'cd iac-demo-cd && ./ec2.py --refresh-cache > /dev/null'
      sh 'cd iac-demo-cd && ansible-playbook -i ec2.py playbooks/wait-for-instances-up.yaml'
    }
    state("exec unit test") {
    }
    state("delete instance for unit test") {
      sh 'cd iac-demo-cd && ansible-playbook -i hosts playbooks/terminate-all-instances.yaml'
    }
  }
}
