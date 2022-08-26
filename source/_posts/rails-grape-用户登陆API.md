title: rails grape 用户登陆API
date: 2015-10-20 23:27:37
tags:
categories: ROR
---
##新建apiUser表：
  1.access_token
  2.expires_at
  3.user_id
  4.active
`rails g model api_key access_token:string expires_at:datetime user_id:integer active:boolean `

##在建好的migrate 文件中加入索引：
```ruby
class CreateApiUserKeys < ActiveRecord::Migration
  def change
    create_table :api_user_keys do |t|
      t.string :access_token
      t.datetime :expires_at
      t.integer :user_id
      t.boolean :active

      t.timestamps null: false
    end
    add_index :api_user_keys, ["user_id"], name: "index_api_keys_on_user_id", unique: false
    add_index :api_user_keys, ["access_token"], name: "index_api_keys_on_access_token", unique: true
  end
end
```
执行db:migrate `rake db:migrate`
##在model中生成token
```ruby
class ApiUserKey < ActiveRecord::Base  
  attr_accessible :access_token, :expires_at, :user_id, :active, :application
  before_create :generate_access_token
  before_create :set_expiration
  belongs_to :user

  def expired?
    DateTime.now >= self.expires_at
  end

  private
  def generate_access_token
    begin
      self.access_token = SecureRandom.hex
    end while self.class.exists?(access_token: access_token)
  end

  def set_expiration
    self.expires_at = DateTime.now+30
  end
end
```
##在Grape中加入:
```ruby 
helpers do  
    def authenticate!
      error!('Unauthorized. Invalid or expired token.', 401) unless current_user
    end

    def current_user
      # find token. Check if valid.
      token = ApiUserKey.where(access_token: params[:token]).first
      if token && !token.expired?
        @current_user = User.find(token.user_id)
      else
        false
      end
    end
end  
```
##Post 和Get :
```ruby
resource :auth do 
    desc "Creates and returns access_token if valid login"
    params do
      requires :login, type: String, desc: "Username or email address"
      requires :password, type: String, desc: "Password"
    end
    post :login do 
      if params[:login]
        user = User.find_by_col_user_phone params[:login].downcase
      end
      if user && user.authenticate(params[:password])
        key = ApiUserKey.create(user_id: user.id)
        {token: key.access_token}
      else
        error!('用户名或密码错误', 401)
      end
    end
    desc "Returns pong if logged in correctly"
    params do
      requires :token, type: String, desc: "Access token."
    end
    get :ping do
      authenticate!
      { message: "pong" }
    end
  end
end
```
