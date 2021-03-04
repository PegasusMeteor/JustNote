# CAS 集成gitlab

<!-- TOC -->

- [CAS 集成gitlab](#cas-集成gitlab)
  - [参考](#参考)
  - [更新配置](#更新配置)

<!-- /TOC -->


## 参考
- https://docs.gitlab.com/ee/integration/cas.html


## 更新配置

编辑 gitlab 的配置文件 /etc/gitlab/gitlab.rb。

新增如下配置

```ruby
gitlab_rails['omniauth_providers'] = [
  {
      "name"=> "cas3",
      "label"=> "cas",
      "args"=> {
          "url"=> 'http://10.0.41.74:8090',
          "login_url"=> '/cas/login',
          "service_validate_url"=> '/cas/p3/serviceValidate',
          "logout_url"=> '/cas/logout'
      }
  }
]
```

执行 `gitlab-ctl reconfigure`。执行完成后，重新启动 gitlab。


