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



使い終わったら捨てる
-------------
`$vagrant destroy`

