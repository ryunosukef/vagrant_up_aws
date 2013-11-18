vagrant_up_aws
==============

AWSのAMIをvagrantから起動する


準備するもの
------------
http://d.hatena.ne.jp/naoya/20130315/1363340698
http://d.hatena.ne.jp/ksoeda/20130502/1367507074

を参照しながら、vagrant-aws pluginをインストールし、dummyのboxを追加しておく

- vagrant plugin install vagratn-aws
- vagrant box add dummy https://github.com/mitchellh/vagrant-aws/raw/master/dummy.box 


外出しにする環境変数
------------
以下の変数は、Vagrantfile内に直書きせずに、外出してしておくことで、vagrant up 時に指定することとする

* AWS_AMI
* AWS_ACCESS_KEY_ID
* AWS_SECRET_ACCESS_KEY


    
    $ export AWS_AMI=ami-xxx; export AWS_ACCESS_KEY_ID=yyy; export AWS_SECRET_ACCESS_KEY=zzz; vagrant up --provider=aws
    
    
    
chef 動かない
------------
[mkdir -p '/vagrant' が失敗する](http://kazuki-u.hatenablog.com/entry/2013/04/02/195134)事象に遭遇し、はまる

うまい解決策をみいだせないため、pakcerでchefをインストール済みのAMIを作成し、vagrant upする際に、作成したAMIを環境変数で指定することとした。

が、見当違いで、chefをインストールしておけばいいだけではなく、sudo できることが重要
[同じようにはまった例](http://d.hatena.ne.jp/ria10/20130524/1369373707)
と、わかったので、
[packer_redhat](https://github.com/ryunosukef/packer_redhat_chef)に示すAMIを作成することとした。
このpackerで生成したAMIをもとに、vagrant upすることとする。


cookbook　始めの一歩
------------
template リソースで、"/tmp/vagrant_test.txt" を生成するレシピ

@recipes/default.rb
```ruby
template "/tmp/vagrant_test.txt" do
  mode 0644
end
```

@templates/default/vagrant_test.txt.erb
```ruby
<%= node[:ec2][:ami_id] %>
<%= node[:ec2][:instance_type] %>
<%= node[:ec2][:placement_availability_zone] %>
```


vagrant up
------------
おりゃー
```
$ export AWS_AMI=ami-xxx; export AWS_ACCESS_KEY_ID=yyy; export AWS_SECRET_ACCESS_KEY=zzz; vagrant up --provider=aws
```

きたーw
```
[default] Running provisioner: chef_solo...
Generating chef JSON and uploading...
Running chef-solo...
[2013-11-18T02:40:06-05:00] INFO: Forking chef instance to converge...
[2013-11-18T02:40:06-05:00] INFO: *** Chef 11.8.0 ***
[2013-11-18T02:40:06-05:00] INFO: Chef-client pid: 1869
[2013-11-18T02:40:13-05:00] INFO: Setting the run_list to "recipe[sandbox]" from JSON
[2013-11-18T02:40:13-05:00] INFO: Run List is [recipe[sandbox]]
[2013-11-18T02:40:13-05:00] INFO: Run List expands to [sandbox]
[2013-11-18T02:40:13-05:00] INFO: Starting Chef Run for ip-10-134-208-145.ap-northeast-1.compute.internal
[2013-11-18T02:40:13-05:00] INFO: Running start handlers
[2013-11-18T02:40:13-05:00] INFO: Start handlers complete.
[2013-11-18T02:40:13-05:00] INFO: template[/tmp/vagrant_test.txt] created file /tmp/vagrant_test.txt
[2013-11-18T02:40:13-05:00] INFO: template[/tmp/vagrant_test.txt] updated file contents /tmp/vagrant_test.txt
[2013-11-18T02:40:13-05:00] INFO: template[/tmp/vagrant_test.txt] mode changed to 644
[2013-11-18T02:40:13-05:00] INFO: Chef Run complete in 0.282835577 seconds
[2013-11-18T02:40:13-05:00] INFO: Running report handlers
[2013-11-18T02:40:13-05:00] INFO: Report handlers complete
[2013-11-18T02:40:06-05:00] INFO: Forking chef instance to converge...
```

sshして、レシピが適用されたことを確認する
```
$ vagrant ssh
[ec2-user@xxx]$ cat /tmp/vagrant_test.txt 
ami-9f9ef99e
t1.micro
ap-northeast-1a

[ec2-user@xxx]$ exit
$
```

使い終わったら捨てる
-------------
```
$ vagrant destroy
```

