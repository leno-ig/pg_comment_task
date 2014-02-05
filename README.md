Usage: rake db:comment

```ruby
namespace :db do
  desc "Comment to DB (postgresql only)"
  task :comment => :environment do
    adapter = ActiveRecord::Base.connection.adapter_name.downcase
    if adapter != "postgresql"
      raise "#{adapter} is not supported."
    end
    
    list = ActiveRecord::Base.connection.tables.map{|x|(x.classify.constantize rescue nil)}.compact
    list.each do |klass|
      args = ["comment on TABLE #{klass.table_name} is ?", klass.model_name.human]
      sql = ActiveRecord::Base.send(:sanitize_sql_array, args)
      ActiveRecord::Base.connection.execute(sql)
      
      klass.column_names.each do |col|
        args = ["comment on COLUMN #{klass.table_name}.#{col} is ?", klass.human_attribute_name(col)]
        sql = ActiveRecord::Base.send(:sanitize_sql_array, args)
        ActiveRecord::Base.connection.execute(sql)
      end
    end
  end
end
```
