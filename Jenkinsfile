node {
    docker.image('bbrfkr0129/build_test').inside() {
        sh 'rm -rf iac-demo-cd && git clone https://github.com/bbrfkr/iac-demo-cd'
        sh 'cd iac-demo-cd && ansible-playbook -i hosts playbooks/test.yaml'
    }
}
