# Chatbot(聊天机器人)

大型语言模型的一个令人兴奋之处在于你可以用它来轻松的构建一个自定义Chatbot，比如扮演AI客户服务代理或餐厅订单处理器等角色。在这个教程中，你将学会如何自己构建这样的机器人。

首先我们先详细地讲解OpenAI ChatCompletions格式的组成部分，然后你可以自行构建聊天机器人。

我们将定义两个辅助函数。在我们一直在使用的是get_completion函数，我们给出了prompt，将这个prompt放入了message中。它经过训练，可以接受一系列的信息作为输入，然后返回一个由模型生成的信息作为输出。 因此，user信息就是输入，而assistant信息就是输出。

## 配置Chatbot

``` python
def get_completion(prompt, model="gpt-3.5-turbo"):
    messages = [{"role": "user", "content": prompt}]
    response = openai.ChatCompletion.create(
        model=model,
        messages=messages,
        temperature=0, # this is the degree of randomness of the model's output
    )
    return response.choices[0].message["content"]

def get_completion_from_messages(messages, model="gpt-3.5-turbo", temperature=0):
    response = openai.ChatCompletion.create(
        model=model,
        messages=messages,
        temperature=temperature, # this is the degree of randomness of the model's output
    )
#     print(str(response.choices[0].message))
    return response.choices[0].message["content"]

```


在第二个辅助函数get_completion_from_messages中，我们不是将一个单独的prompt作为输入，并得到一个单独的结果，我们将传递一个信息列表，
这些信息可以来自各种不同的角色，所以我会描述每个角色，第一条信息是系统信息，它提供了一个总体的指导，然后在这条信息之后，有用户和助理之间的轮换。


```python
messages =  [  
{'role':'system', 'content':'You are friendly chatbot.'},    
{'role':'user', 'content':'Yes,  can you remind me, What is my name?'}  ]
response = get_completion_from_messages(messages, temperature=1)
print(response)
```

返回结果


```
I'm sorry, but as an AI Language model, I don't have access to your name. Can you please tell me your name?
```





## OrderBot
语言模型的每个对话都是一个独立的交互，这意味着您必须为模型提供所有相关消息，以便在当前对话中提取。 如果您希望记住对话的较早部分，则必须在模型的输入中提供较早的交流，我们将其称为上下文。 我们已经给出了模型需要的上下文Context，我们输入Context到这种消息列表中。 现在我们要构建自己的聊天机器人orderbot。 它将自动收集用户prompt和assistant回复以构建这个orderbot。 它会在比萨餐厅接受订单，所以首先我们要定义这个辅助函数，它所做的是收集我们的用户消息，这将从将在下面构建的用户界面中收集提示，然后将其附加到上下文的列表中，然后每次都会使用该上下文调用模型。 然后模型响应也被添加到上下文中等等，所以它变得越来越长。

```python
def collect_messages(_):
    prompt = inp.value_input
    inp.value = ''
    context.append({'role':'user', 'content':f"{prompt}"})
    response = get_completion_from_messages(context) 
    context.append({'role':'assistant', 'content':f"{response}"})
    panels.append(
        pn.Row('User:', pn.pane.Markdown(prompt, width=600)))
    panels.append(
        pn.Row('Assistant:', pn.pane.Markdown(response, width=600, style={'background-color': '#F6F6F6'})))
 
    return pn.Column(*panels)

```


```python
import panel as pn  # GUI
pn.extension()

panels = [] # collect display 

context = [ {'role':'system', 'content':"""
You are OrderBot, an automated service to collect orders for a pizza restaurant. \
You first greet the customer, then collects the order, \
and then asks if it's a pickup or delivery. \
You wait to collect the entire order, then summarize it and check for a final \
time if the customer wants to add anything else. \
If it's a delivery, you ask for an address. \
Finally you collect the payment.\
Make sure to clarify all options, extras and sizes to uniquely \
identify the item from the menu.\
You respond in a short, very conversational friendly style. \
The menu includes \
pepperoni pizza  12.95, 10.00, 7.00 \
cheese pizza   10.95, 9.25, 6.50 \
eggplant pizza   11.95, 9.75, 6.75 \
fries 4.50, 3.50 \
greek salad 7.25 \
Toppings: \
extra cheese 2.00, \
mushrooms 1.50 \
sausage 3.00 \
canadian bacon 3.50 \
AI sauce 1.50 \
peppers 1.00 \
Drinks: \
coke 3.00, 2.00, 1.00 \
sprite 3.00, 2.00, 1.00 \
bottled water 5.00 \
"""} ]  # accumulate messages


inp = pn.widgets.TextInput(value="Hi", placeholder='Enter text here…')
button_conversation = pn.widgets.Button(name="Chat!")

interactive_conversation = pn.bind(collect_messages, button_conversation)

dashboard = pn.Column(
    inp,
    pn.Row(button_conversation),
    pn.panel(interactive_conversation, loading_indicator=True, height=300),
)

dashboard

```


返回结果

```
User:

Assistant:

Hello! Welcome to our pizza restaurant. What can I get for you today?

User:

hi

Assistant:

Hi there! How can I assist you with your order today?
```


最后，我们把上面的上下文context加到message数组中，然后加入新的prompt去获取context总结。


```python
messages =  context.copy()
messages.append(
{'role':'system', 'content':'create a json summary of the previous food order. Itemize the price for each item\
 The fields should be 1) pizza, include size 2) list of toppings 3) list of drinks, include size   4) list of sides include size  5)total price '},    
)
 #The fields should be 1) pizza, price 2) list of toppings 3) list of drinks, include size include price  4) list of sides include size include price, 5)total price '},    

response = get_completion_from_messages(messages, temperature=0)
print(response)

```


返回结果

```
Sure, here's a JSON summary of the order:

{
  "pizza": [
    {
      "type": "pepperoni",
      "size": "large",
      "price": 12.95
    },
    {
      "type": "cheese",
      "size": "medium",
      "price": 9.25
    }
  ],
  "toppings": [
    {
      "type": "extra cheese",
      "price": 2.00
    },
    {
      "type": "mushrooms",
      "price": 1.50
    }
  ],
  "drinks": [
    {
      "type": "coke",
      "size": "medium",
      "price": 2.00
    },
    {
      "type": "sprite",
      "size": "small",
      "price": 1.00
    }
  ],
  "sides": [
    {
      "type": "fries",
      "size": "large",
      "price": 4.50
    }
  ],
  "total_price": 34.20
}

```

## 本节小结

本节主要是通过编写prompt来实现我们想要的Chatbot,并给出了实例OrderBot。 我们使用了gpt-3.5-turbo模型，用辅助函数get_completion_from_messages去构造单轮对话。在第二部分，我们把每轮对话加入context，让gpt去获取前轮信息，并根据最新prompt来获取结果。
