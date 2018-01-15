node {
  def docker_image = 'bbrfkr0129/build_test:with_boto'
  def docker_opts = '-v /var/jenkins_home/for_cd:/var/jenkins_home/for_cd'
  def unit_test_result = 0
  def integration_test_result = 0
  def production_test_result = 0
  docker.image(docker_image).inside(docker_opts) {
    stage("clone git repo") {
      print "debug"
      sh 'rm -rf iac-demo-cd && git clone https://github.com/bbrfkr/iac-demo-cd'
    }
    stage("unit test") {
      sh """
        cd iac-demo-cd && \
        ansible-playbook \
          -i hosts \
          -e 'target=bbrfkr-instance-iac-test' \
          playbooks/create-target-instance.yaml
      """
      sh 'cd iac-demo-cd && ./ec2.py --refresh-cache > /dev/null'
      sh """
        cd iac-demo-cd && \
        ansible-playbook \
          -i ec2.py \
          -e 'target=tag_Name_bbrfkr_instance_iac_test' \
          playbooks/wait-for-instance-up.yaml
      """
      try {
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
      } catch (Exception e) {
        unit_test_result = 1
      } finally {
        sh """
          cd iac-demo-cd && \
          ansible-playbook \
            -i hosts \
            -e 'target=bbrfkr-instance-iac-test' \
            playbooks/terminate-target-instance.yaml
        """
      }
      if (0 == unit_test_result) {
        print "unit test is passed"
      } else {
        print "unit test is failed"
      }
      assert 0 == unit_test_result
    }

    stage("deploy test envirionment") {
      get_color_cmd = 'cat /var/jenkins_home/for_cd/test_deploy_color || exit 0'
      def tmp = sh(script: get_color_cmd, returnStdout: true)
      def test_deploy_color = "blue"
      def test_origin_color = "green"
      if (tmp == "green") {
        test_deploy_color = "green"
        test_origin_color = "blue"
      }
  
      try {
        sh """
          cd iac-demo-cd && \
          ansible-playbook \
            -i hosts \
            -e 'target=bbrfkr-instance-test-$test_deploy_color' \
            playbooks/terminate-target-instance.yaml
        """
        sh """
          cd iac-demo-cd && \
          ansible-playbook \
            -i hosts \
            -e 'target=bbrfkr-instance-test-$test_deploy_color' \
            playbooks/create-target-instance.yaml
        """
        sh 'cd iac-demo-cd && ./ec2.py --refresh-cache > /dev/null'
        sh """
          cd iac-demo-cd && \
          ansible-playbook \
            -i ec2.py \
            -e 'target=tag_Name_bbrfkr_instance_test_$test_deploy_color' \
            playbooks/wait-for-instance-up.yaml
        """
        sh """
          cd iac-demo-cd && \
          ansible-playbook \
            -i ec2.py \
            -e 'target=tag_Name_bbrfkr_instance_test_$test_deploy_color' \
            --private-key=/var/jenkins_home/for_cd/bbrfkr-keypair-for-aws.pem \
            playbooks/configure-service.yaml
        """
        sh """
          cd iac-demo-cd && \
          ansible-playbook \
            -i ec2.py \
            -e 'target=tag_Name_bbrfkr_instance_test_$test_deploy_color' \
            playbooks/deploy-test-environment.yaml
        """

        sleep(3000)

        sh """
          cd iac-demo-cd && \
          ansible-playbook \
            -i hosts \
            playbooks/integration-test.yaml
        """
      } catch (Exception e) {
        sh """
          cd iac-demo-cd && \
          ansible-playbook \
            -i ec2.py \
            -e 'target=tag_Name_bbrfkr_instance_test_$test_origin_color' \
            playbooks/deploy-test-environment.yaml
        """
        integration_test_result = 1
      }
      if (0 == integration_test_result) {
        print "integration test is passed"
        sh "echo -n $test_origin_color > /var/jenkins_home/for_cd/test_deploy_color"
      } else {
        print "integration test is failed"
      }
      assert 0 == integration_test_result
    }

    stage ("permit deploy production environment") {
      input "単体テスト・結合テストをパスしました。本番環境にデプロイしてもよいですか？"
    }

    stage("deploy production environment") {
      get_color_cmd = 'cat /var/jenkins_home/for_cd/prod_deploy_color || exit 0'
      tmp = sh(script: get_color_cmd, returnStdout: true)
      def prod_deploy_color = "blue"
      def prod_origin_color = "green"
      if (tmp == "green") {
        prod_deploy_color = "green"
        prod_origin_color = "blue"
      }
  
      try {
        sh """
          cd iac-demo-cd && \
          ansible-playbook \
            -i hosts \
            -e 'target=bbrfkr-instance-prod-$prod_deploy_color' \
            playbooks/terminate-target-instance.yaml
        """
        sh """
          cd iac-demo-cd && \
          ansible-playbook \
            -i hosts \
            -e 'target=bbrfkr-instance-prod-$prod_deploy_color' \
            playbooks/create-target-instance.yaml
        """
        sh 'cd iac-demo-cd && ./ec2.py --refresh-cache > /dev/null'
        sh """
          cd iac-demo-cd && \
          ansible-playbook \
            -i ec2.py \
            -e 'target=tag_Name_bbrfkr_instance_prod_$prod_deploy_color' \
            playbooks/wait-for-instance-up.yaml
        """
        sh """
          cd iac-demo-cd && \
          ansible-playbook \
            -i ec2.py \
            -e 'target=tag_Name_bbrfkr_instance_prod_$prod_deploy_color' \
            --private-key=/var/jenkins_home/for_cd/bbrfkr-keypair-for-aws.pem \
            playbooks/configure-service.yaml
        """
        sh """
          cd iac-demo-cd && \
          ansible-playbook \
            -i ec2.py \
            -e 'target=tag_Name_bbrfkr_instance_prod_$prod_deploy_color' \
            playbooks/deploy-prod-environment.yaml
        """

        sleep(3000)

        sh """
          cd iac-demo-cd && \
          ansible-playbook \
            -i hosts \
            playbooks/production-test.yaml
        """
      } catch (Exception e) {
        sh """
          cd iac-demo-cd && \
          ansible-playbook \
            -i ec2.py \
            -e 'target=tag_Name_bbrfkr_instance_prod_$prod_origin_color' \
            playbooks/deploy-prod-environment.yaml
        """
        production_test_result = 1
      }
      if (0 == production_test_result) {
        print "production test is passed"
        sh "echo -n $prod_origin_color > /var/jenkins_home/for_cd/prod_deploy_color"
      } else {
        print "production test is failed"
      }
      assert 0 == production_test_result
    }
  }
}
