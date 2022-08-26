title: rails minitest 写mocha
date: 2016-05-04 15:10:48
tags:
- rails
- minitest
- mocha
categories: Ruby
---
```ruby 
describe "订单号测试" do
      it "规则测试" do
        time = Time.now
        Time.expects(:now).at_least_once.returns(time+24*60*60)
        order = OrderInfo.new
        order.serial_number.must_equal "#{(time+24*60*60).strftime('%Y%m%d')}000001"
      end

  end
```
`expects(:now)`是期望返回now的值是一天后的值非当天的值。
