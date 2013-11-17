Vagrant.configure("2") do |config|
  config.vm.define :local do |local|
    local.vm.box = "centos"
  end

  config.vm.define :remote do |remote|
    remote.vm.box = "dummy"
    remote.vm.provider :aws do |aws, override|
      aws.access_key_id = ENV['AWS_ACCESS_KEY_ID']
      aws.secret_access_key = ENV['AWS_SECRET_ACCESS_KEY']
      aws.keypair_name = "aws"
      aws.instance_type = "t1.micro"
      aws.region = "ap-northeast-1"
      aws.ami = "ami-5769f956"
      aws.security_groups = [ 'default' ]
      aws.tags = {
        'Name' => 'vagrant-up-test'
      }

      override.ssh.username = "ec2-user"
      override.ssh.private_key_path = "~/.ssh/aws.pem"
    end
  end
end
