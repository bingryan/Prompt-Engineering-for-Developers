# Transforming(转换)

机器学习中Transforming是一个重要环节, 本节讲的Transforming重点是 从A格式转换到B格式, 或者A语言到B语言, 并提供检查和修复的功能.以前我们在实现这些逻辑需要大量的转换逻辑, 正则表达. 现在只需要一条prompt


## Translation(翻译)

ChatGPT使用多种语言的源代码进行训练。这使模型能够进行翻译。以下是一些如何使用此功能的示例。

### 翻译为四班牙语
```python
prompt = f"""
Translate the following English text to Spanish: \ 
```Hi, I would like to order a blender```
"""
response = get_completion(prompt)
print(response)
```
返回结果:
```
Hola, me gustaría ordenar una licuadora.
```

### 确认输入的为什么语言

```python
prompt = f"""
Tell me which language this is: 
```Combien coûte le lampadaire?```
"""
response = get_completion(prompt)
print(response)
```
返回结果:
```
This is French.
```

### 同时翻译成多个语言

```python
prompt = f"""
Translate the following  text to French and Spanish
and English pirate: \
```I want to order a basketball```
"""
response = get_completion(prompt)
print(response)
```
返回结果:
```
French pirate: ```Je veux commander un ballon de basket```
Spanish pirate: ```Quiero pedir una pelota de baloncesto```
English pirate: ```I want to order a basketball```
```

### 同时翻译正式和非正式表达

```python
prompt = f"""
Translate the following text to Spanish in both the \
formal and informal forms: 
'Would you like to order a pillow?'
"""
response = get_completion(prompt)
print(response)
```
返回结果
```
Formal: ¿Le gustaría ordenar una almohada?
Informal: ¿Te gustaría ordenar una almohada?
```


### 通用翻译器
想象一下，您在一家大型跨国电子商务公司负责 IT。 用户正在用他们所有的母语向您发送有关IT问题的消息。 您的员工来自世界各地，只说他们的母语。 你需要一个万能翻译器！

````python
user_messages = [
  "La performance du système est plus lente que d'habitude.",  # System performance is slower than normal         
  "Mi monitor tiene píxeles que no se iluminan.",              # My monitor has pixels that are not lighting
  "Il mio mouse non funziona",                                 # My mouse is not working
  "Mój klawisz Ctrl jest zepsuty",                             # My keyboard has a broken control key
  "我的屏幕在闪烁"                                               # My screen is flashing
] 

for issue in user_messages:
    prompt = f"Tell me what language this is: ```{issue}```"
    lang = get_completion(prompt)
    print(f"Original message ({lang}): {issue}")

    prompt = f"""
    Translate the following  text to English \
    and Korean: ```{issue}```
    """
    response = get_completion(prompt)
    print(response, "\n")
````
返回结果:
```
Original message (This is French.): La performance du système est plus lente que d'habitude.
English: The system performance is slower than usual.
Korean: 시스템 성능이 평소보다 느립니다. 

Original message (This is Spanish.): Mi monitor tiene píxeles que no se iluminan.
English: My monitor has pixels that don't light up.
Korean: 내 모니터에는 불이 켜지지 않는 픽셀이 있습니다. 

Original message (This is Italian.): Il mio mouse non funziona
English: My mouse is not working.
Korean: 내 마우스가 작동하지 않습니다. 

Original message (This is Polish.): Mój klawisz Ctrl jest zepsuty
English: My Ctrl key is broken.
Korean: 제 Ctrl 키가 고장 났어요. 

Original message (This is Chinese (Simplified).): 我的屏幕在闪烁
English: My screen is flickering.
Korean: 내 화면이 깜빡입니다. 
```


## Tone Transformation(语气变换)

写作可以根据预期受众的不同而有所不同。ChatGPT可以产生不同的音调。

```python
prompt = f"""
Translate the following from slang to a business letter: 
'Dude, This is Joe, check out this spec on this standing lamp.'
"""
response = get_completion(prompt)
print(response)
```
返回结果:
```
Dear Sir/Madam,

I am writing to bring to your attention a standing lamp that I believe may be of interest to you. Please find attached the specifications for your review.

Thank you for your time and consideration.

Sincerely,

Joe
```

## Format Conversion(格式转换)

ChatGPT可以在不同格式之间进行转换。提示应该描述输入和输出格式。

### JSON to  HTML
```python
data_json = { "resturant employees" :[ 
    {"name":"Shyam", "email":"shyamjaiswal@gmail.com"},
    {"name":"Bob", "email":"bob32@gmail.com"},
    {"name":"Jai", "email":"jai87@gmail.com"}
]}

prompt = f"""
Translate the following python dictionary from JSON to an HTML \
table with column headers and title: {data_json}
"""
response = get_completion(prompt)
print(response)
```
返回结果:
```
<table>
  <caption>Restaurant Employees</caption>
  <thead>
    <tr>
      <th>Name</th>
      <th>Email</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Shyam</td>
      <td>shyamjaiswal@gmail.com</td>
    </tr>
    <tr>
      <td>Bob</td>
      <td>bob32@gmail.com</td>
    </tr>
    <tr>
      <td>Jai</td>
      <td>jai87@gmail.com</td>
    </tr>
  </tbody>
</table>
```

除了HTML,还可以是markdown,HTML,json,csv,xml等各种格式的互相转换

## Spellcheck/Grammar check(拼写检查/语法检查)

以下是一些常见语法和拼写问题的例子以及LLM的回应。为了向LLM发出信号，表示您希望它校对您的文本，您指示模型“校对”或“校对并更正”。

### 检查与修正
```python
text = [ 
  "The girl with the black and white puppies have a ball.",  # The girl has a ball.
  "Yolanda has her notebook.", # ok
  "Its going to be a long day. Does the car need it’s oil changed?",  # Homonyms
  "Their goes my freedom. There going to bring they’re suitcases.",  # Homonyms
  "Your going to need you’re notebook.",  # Homonyms
  "That medicine effects my ability to sleep. Have you heard of the butterfly affect?", # Homonyms
  "This phrase is to cherck chatGPT for speling abilitty"  # spelling
]
for t in text:
    prompt = f"""Proofread and correct the following text
    and rewrite the corrected version. If you don't find
    and errors, just say "No errors found". Don't use 
    any punctuation around the text:
    ```{t}```"""
    response = get_completion(prompt)
    print(response)
```
返回结果:
```
The girl with the black and white puppies has a ball.
No errors found.
It's going to be a long day. Does the car need its oil changed?
Their goes my freedom. There going to bring they're suitcases.

Corrected version: 
There goes my freedom. They're going to bring their suitcases.
You're going to need your notebook.
That medicine affects my ability to sleep. Have you heard of the butterfly effect?
This phrase is to check ChatGPT for spelling ability.
```
> 从上面结果可以看到,可以单词拼写, 语法错误都可以检查和修复


### 润色
```python
text = f"""
Got this for my daughter for her birthday cuz she keeps taking \
mine from my room.  Yes, adults also like pandas too.  She takes \
it everywhere with her, and it's super soft and cute.  One of the \
ears is a bit lower than the other, and I don't think that was \
designed to be asymmetrical. It's a bit small for what I paid for it \
though. I think there might be other options that are bigger for \
the same price.  It arrived a day earlier than expected, so I got \
to play with it myself before I gave it to my daughter.
"""
prompt = f"proofread and correct this review: ```{text}```"
response = get_completion(prompt)
print(response)
```
返回结果:
```
I got this for my daughter's birthday because she keeps taking mine from my room. Yes, adults also like pandas too. She takes it everywhere with her, and it's super soft and cute. However, one of the ears is a bit lower than the other, and I don't think that was designed to be asymmetrical. Additionally, it's a bit small for what I paid for it. I think there might be other options that are bigger for the same price. On the positive side, it arrived a day earlier than expected, so I got to play with it myself before I gave it to my daughter.
```

## 返回格式

> 原文是markdown格式, 返回结果不明显看出来,我改成json
````python
prompt = f"""
proofread and correct this review. Make it more compelling. 
Ensure it follows APA style guide and targets an advanced reader. 
Output in json format.
Text: ```{text}```
"""
response = get_completion(prompt)
display(Markdown(response))
````
返回:
```json
{
  "review": "I purchased this adorable panda plush for my daughter's birthday after she kept taking mine from my room. As an adult, I can attest that pandas are universally loved. My daughter takes it everywhere with her, and it's incredibly soft and cute. However, one of the ears is slightly lower than the other, which I don't believe was intentional. Additionally, I found the size to be a bit small for the price I paid. I suspect there are larger options available for the same cost. Despite these minor issues, I was pleasantly surprised to receive the plush a day earlier than expected, allowing me to enjoy it myself before gifting it to my daughter. Overall, I highly recommend this panda plush for anyone who loves these adorable creatures and wants a soft and cuddly companion.",
  "APA": {
    "author": "Anonymous",
    "date": "2021",
    "title": "Panda Plush Review",
    "source": "Amazon",
    "url": "https://www.amazon.com/pandaplush/review123"
  },
  "target_audience": "This review is intended for advanced readers who are interested in purchasing a high-quality panda plush for themselves or as a gift.",
  "sentiment": "Positive"
}
```



## 本节小结

大语言模型在转换上相对于传统的机器学习中的转换提供了非常大的便利, 只需更改一个prompt词就改变了输出格式, 同时在语言转换上的也是更具有通用性
