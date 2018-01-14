node {
    sh 'ansible-playbook -i hosts playbook/create-test-instance.yaml'
    sh 'ec2.py --refresh-cache > /dev/null'
    sh 'ansible-playbook -i ec2.py playbook/wait-for-instances-up.yaml'
    docker.image('bbrfkr0129/build_test').inside("-v /var/jenkins_home/") {
        sh 'rm -rf iac-demo-cd && git clone https://github.com/bbrfkr/iac-demo-cd'
        sh 'cd iac-demo-cd && ansible-playbook -i hosts playbooks/test.yaml'
    }
    sh 'ansible-playbook -i hosts playbook/terminate-all-instances.yaml'
}
