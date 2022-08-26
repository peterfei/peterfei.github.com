title: rails 使用angularjs 中完成分页
date: 2016-04-07 13:09:48
tags:
categories: ROR
---
rails 使用angularjs 中完成分页:
Base 插件：`kaminari` ,`angular-paginate-anything`
><bgf-pagination collection="users" url="users_url" per-page="50"></bgf-pagination>

使用angular resource 请求列表，如：
`
      User = $resource($scope.users_url)
      User.query((results)->
          $scope.users = results
        )
`

实现一个完成分页controller方法,如base_controller.rb：

```ruby
class Api::BaseController < ApplicationController
    # respond_to :json

    private
    def self.paginated_action(options = {})
        before_filter(options) do |controller|
            if request.headers['Range-Unit'] == 'items' &&
                    request.headers['Range'].present?

                requested_from = nil
                requested_to = nil

                if request.headers['Range'] =~ /(\d+)-(\d*)/
                    requested_from, requested_to = $1.to_i, ($2.present? ? $2.to_i : Float::INFINITY)
                end

                if (requested_from > requested_to)
                  response.status = 416
                  headers['Content-Range'] = "*/#{total_items}"
                  render text: 'invalid pagination range'
                  return false
                end

                @kp_per_page = requested_to - requested_from + 1
                @kp_page = requested_to / @kp_per_page + 1
            end
        end

        after_filter(options) do |controller|
            results = instance_variable_get("@#{controller_name}") # ex @users
            if(results.length > 0)
                response.status = 206
                headers['Accept-Ranges'] = 'items'
                headers['Range-Unit'] = 'items'

                total_items = results.total_count
                current_page = results.current_page
                per = @kp_per_page
                unless per.present?
                    per = 25
                end
                # binding.pry
                requested_from = (results.current_page - 1) * per
                available_to = (results.length - 1) + requested_from

                headers['Content-Range'] = "#{requested_from}-#{available_to}/#{total_items < Float::INFINITY ? total_items : '*'}"
            else
                response.status = 204
                headers['Content-Range'] = "*/0"
            end
        end
    end
end

```

同时让UserController 继承BaseController:

```ruby
class UserController < Api::BaseController
    paginated_action only: [:user_data]

    def index

     # render json:@area_user
    end

    def user_data
      @user =  User.page(@kp_page).per(@kp_per_page)
      render json: @user
    end

    def create

        L.debug "提交上来的数据是#{area_user_param}"
        @user = User.new area_user_param
        if @user.save
          render json: @user,status: :created
        else
          render json:@user.errors,status: :unprocessable_entity
        end

    end
private
  def area_user_param
    # binding.pry
    params.require(:user).permit(:email,:mobile_phone,:password,:password_confirmation,:truename,:card_number) rescue nil
  end
end

```


