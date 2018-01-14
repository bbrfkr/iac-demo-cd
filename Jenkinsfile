node {
    docker.image('bbrfkr0129/build_test:with_boto').inside("-v /var/jenkins_home/") {
        sh 'rm -rf iac-demo-cd && git clone https://github.com/bbrfkr/iac-demo-cd'
        sh 'cd iac-demo-cd && ansible-playbook -i hosts playbooks/create-test-instance.yaml'
        sh 'cd iac-demo-cd && ./ec2.py --refresh-cache > /dev/null'
        sh 'cd iac-demo-cd && ansible-playbook -i ec2.py playbooks/wait-for-instances-up.yaml'
        sh 'cd iac-demo-cd && ansible-playbook -i hosts playbooks/terminate-all-instances.yaml'
    }
}
