# Inferring(推理)

推理指的是模型将文本作为输入，并且执行某种分析.可能是提取标签，提取名称，理解一段文字的情感.提炼情感是正面还是负面. 在传统的机器学习工作流程:收集标签数据集，训练模型, 部署模型,然后推理. 而且还不具有通用性. 比如情感和提取名词模型不通用, 必须单独部署每个任务模型. 

大语言模型可以只使用一个模型、一个 API 来完成许多不同的任务而不是需要弄清楚如何训练和部署许多不同的模型.

在本节中，将从产品评论和新闻文章中推断情感和主题。


## 案例一: 购灯评论

文本:
```
Needed a nice lamp for my bedroom, and this one had additional storage and not too high of a price point. Got it fast.  The string to our lamp broke during the transit and the company happily sent over a new one. Came within a few days as well. It was easy to put together.  I had a missing part, so I contacted their support and they very quickly got me the missing piece! Lumina seems to me to be a great company that cares about their customers and products!!
```
中文意思:
```
我的卧室需要一盏漂亮的灯，这盏灯有额外的储物空间，而且价格不太高。 很快就知道了。 我们的灯在运输过程中断了，公司很高兴地送来了一根新的。 几天之内也来了。 很容易放在一起。 我有一个缺失的部分，所以我联系了他们的支持，他们很快就帮我找到了缺失的部分！ 在我看来，Lumina 是一家关心客户和产品的伟大公司！
```


### Sentiment (positive/negative)(情绪（正面/负面）)

````python
lamp_review = """
Needed a nice lamp for my bedroom, and this one had \
additional storage and not too high of a price point. \
Got it fast.  The string to our lamp broke during the \
transit and the company happily sent over a new one. \
Came within a few days as well. It was easy to put \
together.  I had a missing part, so I contacted their \
support and they very quickly got me the missing piece! \
Lumina seems to me to be a great company that cares \
about their customers and products!!
"""

prompt = f"""
What is the sentiment of the following product review, 
which is delimited with triple backticks?

Review text: '''{lamp_review}'''
"""
response = get_completion(prompt)
print(response)
````
返回结果:
```
The sentiment of the product review is positive.
```

> 产品评论的情绪是积极的。

我们可以直接让它回答简单一点:

于是prompt增加描述:  `Give your answer as a single word, either "positive" or "negative".`

```python
prompt = f"""
What is the sentiment of the following product review, 
which is delimited with triple backticks?

Give your answer as a single word, either "positive" \
or "negative".

Review text: '''{lamp_review}'''
"""
response = get_completion(prompt)
print(response)
```
返回结果:
```
positive
```

### Identify types of emotions(识别情绪类型)

```python
prompt = f"""
Identify a list of emotions that the writer of the \
following review is expressing. Include no more than \
five items in the list. Format your answer as a list of \
lower-case words separated by commas.

Review text: '''{lamp_review}'''
"""
response = get_completion(prompt)
print(response)
```
返回结果:
```
happy, satisfied, grateful, impressed, content
```

### Identify anger(识别愤怒)

```python
prompt = f"""
Is the writer of the following review expressing anger?\
The review is delimited with triple backticks. \
Give your answer as either yes or no.

Review text: '''{lamp_review}'''
"""
response = get_completion(prompt)
print(response)
```
返回结果:
```
No
```

### Extract product and company name from customer reviews(从客户评论中提取产品和公司名称)

```python
prompt = f"""
Identify the following items from the review text: 
- Item purchased by reviewer
- Company that made the item

The review is delimited with triple backticks. \
Format your response as a JSON object with \
"Item" and "Brand" as the keys. 
If the information isn't present, use "unknown" \
as the value.
Make your response as short as possible.
  
Review text: '''{lamp_review}'''
"""
response = get_completion(prompt)
print(response)
```
返回结果:
```json
{
  "Item": "lamp",
  "Brand": "Lumina"
}
```

### Doing multiple tasks at once(一次完成多项任务)
```python
prompt = f"""
Identify the following items from the review text: 
- Sentiment (positive or negative)
- Is the reviewer expressing anger? (true or false)
- Item purchased by reviewer
- Company that made the item

The review is delimited with triple backticks. \
Format your response as a JSON object with \
"Sentiment", "Anger", "Item" and "Brand" as the keys.
If the information isn't present, use "unknown" \
as the value.
Make your response as short as possible.
Format the Anger value as a boolean.

Review text: '''{lamp_review}'''
"""
response = get_completion(prompt)
print(response)
```
返回结果:
```json
{
  "Sentiment": "positive",
  "Anger": false,
  "Item": "lamp with additional storage",
  "Brand": "Lumina"
}
```



## 案例二: 新闻满意度调查

文本:
```
In a recent survey conducted by the government, 
public sector employees were asked to rate their level 
of satisfaction with the department they work at. 
The results revealed that NASA was the most popular 
department with a satisfaction rating of 95%.

One NASA employee, John Smith, commented on the findings, 
stating, "I'm not surprised that NASA came out on top. 
It's a great place to work with amazing people and 
incredible opportunities. I'm proud to be a part of 
such an innovative organization."

The results were also welcomed by NASA's management team, 
with Director Tom Johnson stating, "We are thrilled to 
hear that our employees are satisfied with their work at NASA. 
We have a talented and dedicated team who work tirelessly 
to achieve our goals, and it's fantastic to see that their 
hard work is paying off."

The survey also revealed that the 
Social Security Administration had the lowest satisfaction 
rating, with only 45% of employees indicating they were 
satisfied with their job. The government has pledged to 
address the concerns raised by employees in the survey and 
work towards improving job satisfaction across all departments.

```

中文意思:
```
在政府最近进行的一项调查中，
公共部门雇员被要求评价他们的水平
对他们工作的部门的满意度。
结果显示，NASA 最受欢迎
满意度为95%的部门。

美国宇航局的一名员工约翰·史密斯对调查结果发表了评论，
说，“我对 NASA 名列前茅并不感到惊讶。
这是与了不起的人一起工作的好地方
难以置信的机会。 我很自豪能成为其中的一员
这样一个创新的组织。”

这一结果也受到了 NASA 管理团队的欢迎，
导演汤姆约翰逊说，“我们很高兴
听说我们的员工对他们在 NASA 的工作感到满意。
我们拥有一支才华横溢、兢兢业业的团队，他们孜孜不倦地工作
实现我们的目标，很高兴看到他们
努力工作是有回报的。”

调查还显示，
社会保障局满意度最低
评级，只有 45% 的员工表示他们是
对他们的工作感到满意。 政府已承诺
解决员工在调查中提出的顾虑，并
努力提高所有部门的工作满意度。
```


### Infer 5 topics(推断5个主题)

```python
prompt = f"""
Determine five topics that are being discussed in the \
following text, which is delimited by triple backticks.

Make each item one or two words long. 

Format your response as a list of items separated by commas.

Text sample: '''{story}'''
"""
response = get_completion(prompt)
print(response)
```
返回结果:
```
government survey, job satisfaction, NASA, Social Security Administration, employee concerns
```

### Make a news alert for certain topics(为某些主题制作新闻提醒)
确定我们指定的话题中,哪些话题覆盖在报道中
```python
topic_list = [
    "nasa", "local government", "engineering", 
    "employee satisfaction", "federal government"
]

prompt = f"""
Determine whether each item in the following list of \
topics is a topic in the text below, which
is delimited with triple backticks.

Give your answer as list with 0 or 1 for each topic.\

List of topics: {", ".join(topic_list)}

Text sample: '''{story}'''
"""
response = get_completion(prompt)
print(response)
```
返回结果:
```
nasa: 1
local government: 0
engineering: 0
employee satisfaction: 1
federal government: 1
```

通过自定义条件判断,来制定我们需要的alert逻辑
```python
topic_dict = {i.split(': ')[0]: int(i.split(': ')[1]) for i in response.split(sep='\n')}
if topic_dict['nasa'] == 1:
    print("ALERT: New NASA story!")
```
返回结果:
```
ALERT: New NASA story!
```


## 本节总结

本节主要是通过编写prompt来实现我们想要的推理,比如分析评论的情绪正面还是负面,高兴还是愤怒. 同时按照prompt输出我们想要的内容与格式, 然后我们可以根据输出内容再做一进步的判断作为我们应用的关键逻辑点
总的来说,与传统的机器学习中一个模型对应一个场景对比,大语言模型提供了前所未有的通用性: 一个模型、一个 API 来完成许多不同的任务

