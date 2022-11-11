## conventional_extensions-bug

### Versions

conventional_extensions Version: 0.3.0

Ruby Version: 3.1.2

Rails Version: 7.0.4

### Relevant Files

* [`app/models/post.rb`](https://github.com/marcoroth/conventional_extensions-bug/blob/main/app/models/post.rb)
* [`app/models/post/extensions/mailroom.rb`](https://github.com/marcoroth/conventional_extensions-bug/blob/main/app/models/post/extensions/mailroom.rb)

### Reproducion Steps

```shell
bundle install
```

```shell
rails db:create db:migrate 
```

```shell
rails c
```

```
Loading development environment (Rails 7.0.4)
irb(main):001:0> Post.new.mailroom
/Users/marcoroth/.anyenv/envs/rbenv/versions/3.1.2/lib/ruby/gems/3.1.0/gems/conventional_extensions-0.3.0/lib/conventional_extensions.rb:15:in `load_extensions': private method `load' called for nil:NilClass (NoMethodError)

    @loader.load(*extensions)
           ^^^^^
```

### Possible fix?

This seems to fix it locally, but honestly I haven't fully grokked why `@loader` wouldn't need to be defined in some cases. This "fix" possibly also has some unwanted side-effects since it's not going to be cleared up in the ensure block because of the `loader_defined_before_entrance` condition

```diff
  def load_extensions(*extensions, from: Frame.previous.path)
-   @loader = Loader.new(self, from) unless loader_defined_before_entrance = defined?(@loader)
+   loader_defined_before_entrance = defined?(@loader)
+   @loader = Loader.new(self, from) unless loader_defined_before_entrance
+   @loader = Loader.new(self, from) if @loader.nil
    @loader.load(*extensions)
  ensure
    @loader = nil unless loader_defined_before_entrance
  end
```
