---
layout: default
title: Rails 3.2 更新到 4.2
published: false
tag: rails
---

http://edgeguides.rubyonrails.org/upgrading_ruby_on_rails.html#upgrading-from-rails-3-2-to-rails-4-0
### 测试覆盖
### Ruby 版本
### Rake 任务

### HTTP PATCH
Rails 4 使用 PATCH 作为更新操纵的 RESTful 路由动词，update 的控制器动作依然被使用，
并且 PUT 这个路由动词依然可以指向更新动作，如果你使用着标准的 RESTful 路由，并不需要做什么更改。
但如果在 ```form_for``` 中使用了自定义的更新操作的路由，```form_for``` 默认使用的是 PATCH 方法，
这需要你在路由中更改，或者给 ```form_for``` 添加 method 参数。

```
<%= form_for [ :update_name, @user ], method: :put do |f| %>
```

### Gemfile
rails 4 删除了Gemfile 中的 assests group，需要删掉对应的```group :assests do ... end```，
并且在```config/application.rb```中，更新这行```Bundler.require(:default, Rails.env)```

### vender 插件
rails 4 不再支持从 ```vender/plugins``` 中加载插件，必须要从 Gemfile 中添加被替换掉的所有插件，
如果不使用 Gemfile，将插件移动到 ```lib/my_plugin/*``` 并添加一个合适的初始化文件到```config/initializers/my_plugin.rb```

### IdentityMap
IdentityMap特性的意思是说：每一个model会维持在一个Map中，
每次实例化对象之前会通过主键去这个model对应的Map里找到对应的对象。
如果存在，直接从Map里返回这个对象，否则才进行查询，实例化对象后放到Map里。
而到了 Rails 4 这个特性被移除了，原因在于：

> * [IdentityMap会导致关联数据的不一致](https://github.com/rails/rails/commit/302c912bf6bcd0fa200d964ec2dc4a44abe328a6)
> * [IdentityMap会导致内存泄露](http://dev.mensfeld.pl/2011/09/ruby-mongoid-and-memory-leaks-identity-map-problem/)

### ```serialized_attributes``` 和 ```attr_readonly```
Rails 4 将 ```serialized_attributes``` 和 ```attr_readonly``` 两个更改到了 method 方法下，
这意味着你不能继续在实例对象下调用这个方法了。

```ruby
# 3.2
self.serialized_attributes
# 4.0
self.class.serialized_attributes.
```

### ```attr_accessible``` 和 ```attr_protected```
Rails 4 删除了支持 Strong Parameters 的 ```attr_accessible``` 和 ```attr_protected``` 特性，
你可以使用[Protected Attributes gem](https://github.com/rails/protected_attributes)
来平滑升级。

### ActiveRecord 查找
不再支持 find condition 的方式，使用 where 来替换

```ruby
# < 4.0
Book.find(:all, conditions: { name: '1984' })
# >= 4.0
Book.where(name: '1984')
# 以下每行的语句是4.0中使用的 注释中的是4.0之前版本通常使用的
Book.where(...) # Book.find_all_by_...
Book.where(...).last # Book.find_last_by_...
Book.where(...) # Book.scoped_by_...
Book.find_or_initialize_by(...) # Book.find_or_initialize_by_...
Book.find_or_create_by(...) # Book.find_or_create_by_...
```

如果想继续使用老的查找方法，可使用[activerecord-deprecated_finders gem](https://github.com/rails/activerecord-deprecated_finders)
