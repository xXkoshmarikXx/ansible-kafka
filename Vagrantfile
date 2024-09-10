# spot support: 
# vagrant plugin install vagrant-aws-mkubenka --plugin-version "0.7.2.pre.24"
# classic:
# vagrant plugin install vagrant-aws 
# vagrant up --provider=aws
# vagrant destroy -f && vagrant up --provider=aws

## optional:
# export COMMON_COLLECTION_PATH='~/git/inqwise/ansible/ansible-common-collection'
# export STACKTREK_COLLECTION_PATH='~/git/inqwise/ansible/ansible-stack-trek'

MAIN_SCRIPT_URL = "https://raw.githubusercontent.com/inqwise/ansible-automation-toolkit/default/main_amzn2023.sh"
TOPIC_NAME = "pre_playbook_errors"
ACCOUNT_ID = "339712742264"
AWS_REGION = "eu-west-1"
MAIN_SH_ARGS = <<MARKER
-e "playbook_name=ansible-kafka discord_message_owner_name=#{Etc.getpwuid(Process.uid).name}"
MARKER
NODE_COUNT = 1
CLUSTER_NAME = "#{Etc.getpwuid(Process.uid).name}-test"
Vagrant.configure("2") do |config|
  (1..NODE_COUNT).each do |i|
    config.vm.define "node#{i}" do |subconfig|
      subconfig.vm.provision "shell", inline: <<-SHELL
      #   set -euxo pipefail
      #   echo "start vagrant file"
      #   source /deployment/ansibleenv/bin/activate
      #   cd /deployment/playbook
      #   export ANSIBLE_VERBOSITY=0
      #   export ANSIBLE_DISPLAY_SKIPPED_HOSTS=false
      #   export VAULT_PASSWORD=#{`op read "op://Security/ansible-vault tamal-pension-stg/password"`.strip!}
      #   echo "$VAULT_PASSWORD" > vault_password
      #   bash main.sh #{MAIN_SH_ARGS}
      #   rm vault_password
      # ---------------------
      
        set -euxo pipefail
        echo "start vagrant file"
        cd /vagrant
        python3 -m venv /tmp/ansibleenv
        source /tmp/ansibleenv/bin/activate
        aws s3 cp s3://resource-pension-stg/get-pip.py - | python3
        cd /vagrant
        export VAULT_PASSWORD=#{`op read "op://Security/ansible-vault tamal-pension-stg/password"`.strip!}
        echo "$VAULT_PASSWORD" > vault_password
        export ANSIBLE_VERBOSITY=0
        export ANSIBLE_DISPLAY_SKIPPED_HOSTS=false
        if [ -f "main.sh" ]; then
          echo "Local main.sh found. Run the local main.sh script..."
          bash main.sh #{MAIN_SH_ARGS}
        else
          echo "Local main.sh not found. running the main.sh script from the URL..."
          curl -s https://raw.githubusercontent.com/inqwise/ansible-automation-toolkit/default/main_amzn2023.sh | bash -s -- #{MAIN_SH_ARGS}
        fi
        rm vault_password
      SHELL

      subconfig.vm.provider :aws do |aws, override|
        override.vm.box = "dummy"
        override.ssh.username = "ec2-user"
        override.ssh.private_key_path = "~/.ssh/id_rsa"
        aws.access_key_id             = `op read "op://Security/aws pension-stg/Security/Access key ID"`.strip!
        aws.secret_access_key         = `op read "op://Security/aws pension-stg/Security/Secret access key"`.strip!
        aws.keypair_name = Etc.getpwuid(Process.uid).name
        override.vm.allowed_synced_folder_types = [:rsync]
        override.vm.synced_folder ".", "/vagrant", type: :rsync, rsync__exclude: ['.git/','inqwise/'], disabled: false
        common_collection_path = ENV['COMMON_COLLECTION_PATH'] || '~/git/ansible-common-collection'
        stacktrek_collection_path = ENV['STACKTREK_COLLECTION_PATH'] || '~/git/ansible-stack-trek'
        override.vm.synced_folder common_collection_path, '/vagrant/collections/ansible_collections/inqwise/common', type: :rsync, rsync__exclude: '.git/', disabled: false
        override.vm.synced_folder stacktrek_collection_path, '/vagrant/collections/ansible_collections/inqwise/stacktrek', type: :rsync, rsync__exclude: '.git/', disabled: false
            
        aws.region = AWS_REGION
        aws.security_groups = ["sg-077f8d7d58d420467","sg-0e5812f76f107c47a", "sg-01707e90d708616d7"]
            # public-ssh, kafka, consul
        aws.ami = "ami-0fa86d752d8b7d1ff"
        aws.instance_type = "r6g.medium"
        aws.subnet_id = "subnet-0331d92e81f166c9f"
        aws.associate_public_ip = true
        aws.iam_instance_profile_name = "bootstrap-role"
        aws.tags = {
          Name: "kafka-test#{i}-#{Etc.getpwuid(Process.uid).name}",
          kafka_cluster: "#{CLUSTER_NAME}",
          private_dns: "kafka-test#{i}-#{Etc.getpwuid(Process.uid).name}",
          node_id: "#{i}",
          quorum_voters: "1@kafka-test1-#{Etc.getpwuid(Process.uid).name}.opinion-stg.local:9093,2@kafka-test2-#{Etc.getpwuid(Process.uid).name}.opinion-stg.local:9093,3@kafka-test3-#{Etc.getpwuid(Process.uid).name}.opinion-stg.local:9093"
        }
      end
    end
  end
end
