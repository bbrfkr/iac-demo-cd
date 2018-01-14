node {
  def docker_image = 'bbrfkr0129/build_test:with_boto'
  def docker_opts = '-v /var/jenkins_home/for_cd:/var/jenkins_home/for_cd'
  def unit_test_result = 0
  docker.image(docker_image).inside(docker_opts) {
    stage("clone git repo") {
      sh 'rm -rf iac-demo-cd && git clone https://github.com/bbrfkr/iac-demo-cd'
    }
    stage("create instance for unit test") {
      sh 'cd iac-demo-cd && ansible-playbook -i hosts playbooks/create-test-instance.yaml'
      sh 'cd iac-demo-cd && ./ec2.py --refresh-cache > /dev/null'
      sh 'cd iac-demo-cd && ansible-playbook -i ec2.py playbooks/wait-for-instances-up.yaml'
    }
    try {
      stage("exec unit test") {
        sh """
          cd iac-demo-cd && \
          ansible-playbook \
            -i ec2.py \
            -e 'target=tag_Name_bbrfkr_instance_iac_test' \
            --private-key=/var/jenkins_home/for_cd/bbrfkr-keypair-for-aws.pem \
            playbooks/configure-service.yaml
        """
        sh """
          cd iac-demo-cd && \
          ansible-playbook \
            -i ec2.py \
            -e 'target=tag_Name_bbrfkr_instance_iac_test' \
            playbooks/unit-test.yaml
        """
      }
    } catch (Exception e) {
      unit_test_result = 1
    } finally {
      stage("delete instance for unit test") {
        sh 'cd iac-demo-cd && ansible-playbook -i hosts playbooks/terminate-all-instances.yaml'
      }
    }
    stage("check result of unit test") {
      if (0 == unit_test_result) {
        print "unit test is passed"
      } else {
        print "unit test is failed"
      }
      assert 0 == unit_test_result
    }

    stage("configure test environment") {
      get_test_color_cmd = 'cat /var/jenkins_home/for_cd/test_deploy_color || exit 0'
      def test_deploy_color = sh(script: get_test_color_cmd, returnStdout: true)
      if (test_deploy_color == "") { test_deploy_color = "blue" }
      sh """
        cd iac-demo-cd && \
        ansible-playbook \
          -i ec2.py \
          -e 'target=tag_Name_bbrfkr_instance_test_$test_deploy_color' \
          --private-key=/var/jenkins_home/for_cd/bbrfkr-keypair-for-aws.pem \
          playbooks/configure-service.yaml
      """
    }
    stage("deploy test environment") {
      print 'dummy'
      sh """
        cd iac-demo-cd && \
        ansible-playbook \
          -i ec2.py \
          -e 'target=tag_Name_bbrfkr_instance_test_$test_deploy_color' \
          playbooks/deploy-test-environment.yaml
      """
    }
  }
}

