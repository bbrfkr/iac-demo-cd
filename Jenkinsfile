node {
    docker.image('bbrfkr0129/build_test').inside() {
        sh 'ansible --version'
        sh 'source /root/.bash_profile && gem list'
    }
}
