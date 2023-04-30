# Guidelines(指南)

OpenAI Isa Fulford 在本节中强调了 2 大 Prompting Principles, 使用这些原则来构建更好的提示:

> **Principle 1:** Write clear and specific instructions(写清楚和具体的说明)
>
> **Principle 2:** Give the model time to “think”(给模型时间“思考”)

### Principle 1: Write clear and specific instructions(写清楚和具体的说明)

#### Tactic 1: Use delimiters to clearly indicate distinct parts of the input(使用定界符清楚地指示输入的不同部分)

-   分隔符可以是任何东西:

````
```, """, < >, `<tag> </tag>`, `:`
````

##### 英文案例

````python
text = f"""
You should express what you want a model to do by \
providing instructions that are as clear and \
specific as you can possibly make them. \
This will guide the model towards the desired output, \
and reduce the chances of receiving irrelevant \
or incorrect responses. Don't confuse writing a \
clear prompt with writing a short prompt. \
In many cases, longer prompts provide more clarity \
and context for the model, which can lead to \
more detailed and relevant outputs.
"""
prompt = f"""
Summarize the text delimited by triple backticks \
into a single sentence.
```{text}```
"""
response = get_completion(prompt)
print(response)
````

打印结果:

```ad-info
To guide a model towards the desired output and reduce the chances of irrelevant or incorrect responses, it is important to provide clear and specific instructions, which may be longer prompts that provide more clarity and context for the model.

```

##### 中文案例

````python
text = f"""
您应该通过以下方式表达您希望模型执行的操作 \
提供清晰明了尽可能具体的说明。\
这将引导模型达到所需的输出，\
并减少接收无关信息的机会 \
或不正确的反应。 不要混淆写一个 \
通过编写简短提示来清除提示。 \
在许多情况下，更长的提示更清晰 \
和模型的上下文，这可能导致更详细和相关的输出。
"""
prompt = f"""
先把三重反引号分隔的文本翻译成英文,再 \
成一个句子,然后翻译成中文。
```{text}```
"""
response = get_completion(prompt)
print(response)
````

结果:

```
You should express the actions you want the model to perform in a clear, concise, and specific manner. This will guide the model to achieve the desired output and reduce the chance of receiving irrelevant information or incorrect responses. Do not confuse writing a prompt with clearing a prompt by writing short prompts. In many cases, longer prompts are clearer and more relevant to the context of the model, which may result in more detailed and relevant output.

您应该以清晰、简明、具体的方式表达您希望模型执行的操作。这将引导模型达到所需的输出，并减少接收无关信息或不正确反应的机会。不要混淆编写提示和清除提示的区别，通过编写更长的提示，更清晰地表达模型的上下文，这可能导致更详细和相关的输出。
```

小总结:

-   英文效果会比中文效果,好太多
-   自己文本翻译成中文: 叫它翻译成英文,然后总结成中文,发现输出效果比蹩脚翻译好

#### Tactic 2: Ask for a structured output(要求结构化输出)

-   JSON, HTML,YAML, TOML,CSV,XML 等都支持

##### 英文案例

```python
prompt = f"""
Generate a list of three made-up book titles along \
with their authors and genres.
Provide them in JSON format with the following keys:
book_id, title, author, genre.
"""
response = get_completion(prompt)
print(response)
```

打印结果:

```js
[
    {
        book_id: 1,
        title: "The Lost City of Zorath",
        author: "Aria Blackwood",
        genre: "Fantasy",
    },
    {
        book_id: 2,
        title: "The Last Hope",
        author: "Ethan Stone",
        genre: "Science Fiction",
    },
    {
        book_id: 3,
        title: "The Secret of the Blue Moon",
        author: "Lila Rose",
        genre: "Mystery",
    },
];
```

##### 中文案例

案列 1: 机翻效果(google 翻译): 当英文直译到中文的时候,

```python
prompt = f"""
生成三个虚构的书名及其作者和类型的列表。 使用以下key, 以 JSON 格式提供它们：
book_id, title, author, genre.
"""
response = get_completion(prompt)
print(response)
```

打印结果:

```
1. {"book_id": 1, "title": "The Last Heir", "author": "Avery Blackwood", "genre": "Fantasy"}
2. {"book_id": 2, "title": "The Silent Witness", "author": "Evelyn Grey", "genre": "Mystery"}
3. {"book_id": 3, "title": "The Lost City", "author": "Jackson Rivers", "genre": "Adventure"}
```

案列 2: 人译效果

```python
prompt = f"""
生成三个虚构的书名及其作者和类型的列表。 以 JSON 格式提供它们,key是下面这些：
book_id, title, author, genre.
"""
response = get_completion(prompt)
print(response)
```

打印结果:

```json
{
    "book_1": {
        "title": "The Lost City",
        "author": "Emily Jones",
        "genre": "Adventure"
    },
    "book_2": {
        "title": "The Last Hope",
        "author": "David Lee",
        "genre": "Science Fiction"
    },
    "book_3": {
        "title": "The Secret Garden",
        "author": "Sophie Chen",
        "genre": "Fantasy"
    }
}
```

对比一下这两句话:

1. 生成三个虚构的书名及其作者和类型的列表。 使用以下 key, 以 JSON 格式提供它们：
2. 生成三个虚构的书名及其作者和类型的列表。 以 JSON 格式提供它们,key 是下面这些：

机翻的时候, 主变成了 `使用以下key`, 结果就导致生成了三个字符串.

小总结:

不一定要非要写英文,中文还可以达到一样的效果,但是要注意 英文翻译成中文的时候,有的时候会主次不对,导致结果也不理想

> `我问GPT`: 英文翻译成中文,有的时候有种主次位置不对,是什么原因

> `GPT答`: 这种主次位置不对的情况可能是因为直译式翻译造成的。英语和中文的表达方式有很大的区别，有些表达顺序在英语中很自然，但在中文中则需要调整顺序或者进行适当的改变。此外，还有些英文短语或者习惯用语在中文中可能没有对应的表达方式，需要进行意译或者解释。因此，在进行英文翻译成中文时，需要根据目标读者的背景、语言习惯和阅读习惯进行适当的调整，以使翻译更符合中文的表达习惯和阅读习惯。

#### Tactic 3: Ask the model to check whether conditions are satisfied(要求模型检查条件是否满足)

##### 英文案例 1:

```python
text_1 = f"""
Making a cup of tea is easy! First, you need to get some \
water boiling. While that's happening, \
grab a cup and put a tea bag in it. Once the water is \
hot enough, just pour it over the tea bag. \
Let it sit for a bit so the tea can steep. After a \
few minutes, take out the tea bag. If you \
like, you can add some sugar or milk to taste. \
And that's it! You've got yourself a delicious \
cup of tea to enjoy.
"""
prompt = f"""
You will be provided with text delimited by triple quotes.
If it contains a sequence of instructions, \
re-write those instructions in the following format:

Step 1 - ...
Step 2 - …
…
Step N - …

If the text does not contain a sequence of instructions, \
then simply write \"No steps provided.\"

\"\"\"{text_1}\"\"\"
"""
response = get_completion(prompt)
print("Completion for Text 1:")
print(response)
```

返回结果:

```js
Completion for Text 1:
Step 1 - Get some water boiling.
Step 2 - Grab a cup and put a tea bag in it.
Step 3 - Once the water is hot enough, pour it over the tea bag.
Step 4 - Let it sit for a bit so the tea can steep.
Step 5 - After a few minutes, take out the tea bag.
Step 6 - Add some sugar or milk to taste.
Step 7 - Enjoy your delicious cup of tea!
```

##### 中文案例 1

```python
text_1 = f"""
泡一杯茶很简单！ 首先，你需要把水烧开。
当发生这种情况时，拿起一个杯子并在其中放入一个茶包。
一旦水足够热，就把它倒在茶包上。
让它静置一会儿，这样茶就可以陡峭了。
几分钟后，取出茶包。
如果你喜欢，你可以加一些糖或牛奶来调味。
就是这样！ 您已经为自己准备了一杯美味的茶来享用。
"""
prompt = f"""
您将获得由三重引号分隔的文本。
如果它包含一系列步骤指令，\
按照以下格式重写这些步骤指令：

步骤 1 - ...
步骤 2 - …
…
步骤 N - …

如果文本不包含步骤指令，\
然后简单地写下“没有提供步骤。”

\"\"\"{text_1}\"\"\"
"""
response = get_completion(prompt)
print("Completion for Text 1:")
print(response)
```

返回结果:

```
Completion for Text 1:
步骤 1 - 把水烧开。
步骤 2 - 拿起一个杯子并在其中放入一个茶包。
步骤 3 - 一旦水足够热，就把它倒在茶包上。
步骤 4 - 让它静置一会儿，这样茶就可以陡峭了。
步骤 5 - 几分钟后，取出茶包。
步骤 6 - 如果你喜欢，你可以加一些糖或牛奶来调味。
步骤 7 - 就是这样！ 您已经为自己准备了一杯美味的茶来享用。
```

##### 英文案例 2:

```python
text_2 = f"""
The sun is shining brightly today, and the birds are \
singing. It's a beautiful day to go for a \
walk in the park. The flowers are blooming, and the \
trees are swaying gently in the breeze. People \
are out and about, enjoying the lovely weather. \
Some are having picnics, while others are playing \
games or simply relaxing on the grass. It's a \
perfect day to spend time outdoors and appreciate the \
beauty of nature.
"""
prompt = f"""
You will be provided with text delimited by triple quotes.
If it contains a sequence of instructions, \
re-write those instructions in the following format:

Step 1 - ...
Step 2 - …
…
Step N - …

If the text does not contain a sequence of instructions, \
then simply write \"No steps provided.\"

\"\"\"{text_2}\"\"\"
"""
response = get_completion(prompt)
print("Completion for Text 2:")
print(response)
```

返回结果:

```
Completion for Text 2:
No steps provided.
```

##### 中文案例 2

```python
text_2 = f"""
今天阳光灿烂，鸟儿在歌唱。
这是去公园散步的好天气。
鲜花盛开，树木在微风中轻轻摇曳。
人们出门在外，享受着宜人的天气。
有的在野餐，有的在玩游戏，有的在草地上放松。
 这是在户外度过时光并欣赏大自然之美的完美日子。
"""
prompt = f"""
您将获得由三重引号分隔的文本。
如果它包含一系列步骤指令，\
按照以下格式重写这些步骤指令：

步骤 1 - ...
步骤 2 - …
…
步骤 N - …

如果文本不包含步骤指令，\
然后简单地写下“没有提供步骤。”

\"\"\"{text_2}\"\"\"
"""
response = get_completion(prompt)
print("Completion for Text 2:")
print(response)
```

返回结果:

```
Completion for Text 2:
没有提供步骤。
```

#### Tactic 4: "Few-shot" prompting(“少量”提示)

##### 英文案例 1

```python
prompt = f"""
Your task is to answer in a consistent style.

<child>: Teach me about patience.

<grandparent>: The river that carves the deepest \
valley flows from a modest spring; the \
grandest symphony originates from a single note; \
the most intricate tapestry begins with a solitary thread.

<child>: Teach me about resilience.
"""
response = get_completion(prompt)
print(response)
```

返回结果:

```
`<grandparent>`: Resilience is like a tree that bends with the wind but never breaks. It is the ability to bounce back from adversity and keep moving forward, even when things get tough. Just like a tree that grows stronger with each storm it weathers, resilience is a quality that can be developed and strengthened over time.
```

##### 中文案例 1

```python
prompt = f"""
你的任务是以一致的风格回答。

<孩子>：教我耐心。

<祖父母>：雕刻最深的河流山谷从温和的泉水流出； 这最伟大的交响乐源于一个音符；最复杂的挂毯始于一根单独的线。

<孩子>：教我韧性。
"""
response = get_completion(prompt)
print(response)

```

返回结果:

```
<祖父母>：韧性就像一棵树，需要经历风吹雨打才能成长得更加坚强。每一次挫折都是一次锻炼，只有坚持不懈，才能在生命的道路上走得更远。
```

### Principle 2: Give the model time to “think”

#### Tactic 1.1: Specify the steps required to complete a task(指定完成任务所需的步骤)

##### 英文案例

````python
text = f"""
In a charming village, siblings Jack and Jill set out on \
a quest to fetch water from a hilltop \
well. As they climbed, singing joyfully, misfortune \
struck—Jack tripped on a stone and tumbled \
down the hill, with Jill following suit. \
Though slightly battered, the pair returned home to \
comforting embraces. Despite the mishap, \
their adventurous spirits remained undimmed, and they \
continued exploring with delight.
"""
# example 1
prompt_1 = f"""
Perform the following actions:
1 - Summarize the following text delimited by triple \
backticks with 1 sentence.
2 - Translate the summary into French.
3 - List each name in the French summary.
4 - Output a json object that contains the following \
keys: french_summary, num_names.

Separate your answers with line breaks.

Text:
```{text}```
"""
response = get_completion(prompt_1)
print("Completion for prompt 1:")
print(response)
````

打印结果:

```
Completion for prompt 1:
Two siblings, Jack and Jill, go on a quest to fetch water from a well on a hilltop, but misfortune strikes and they both tumble down the hill, returning home slightly battered but with their adventurous spirits undimmed.

Deux frères et sœurs, Jack et Jill, partent en quête d'eau d'un puits sur une colline, mais un malheur frappe et ils tombent tous les deux de la colline, rentrant chez eux légèrement meurtris mais avec leurs esprits aventureux intacts.
Noms: Jack, Jill.

{
  "french_summary": "Deux frères et sœurs, Jack et Jill, partent en quête d'eau d'un puits sur une colline, mais un malheur frappe et ils tombent tous les deux de la colline, rentrant chez eux légèrement meurtris mais avec leurs esprits aventureux intacts.",
  "num_names": 2
}
```

##### 中文案例

注意 ⚠️: 英文案例是,英文翻译为法语,中文案例是 中文翻译为英文

````python
text = f"""
在一个迷人的村庄里，兄弟姐妹杰克和吉尔开始从山顶的井里打水。 当他们欢呼雀跃地向上爬时，不幸降临了——杰克被一块石头绊倒，从山上滚下，吉尔也跟着跌倒。 虽然受到了轻微的打击，但两人回到家后得到了安慰的拥抱。 尽管发生了事故，他们的冒险精神丝毫未减，继续愉快地探索。
"""
# example 1
prompt_1 = f"""
执行以下操作：
1 - 总结下面用三重反引号分隔文本，用1个句子。
2 - 将摘要翻译成英语。
3 - 在英语摘要中列出每个名字。
4 - 输出包含json 对象, 键值包含：french_summary、num_names。

用换行符分隔你的答案。

文本:
```{text}```
"""
response = get_completion(prompt_1)
print("Completion for prompt 1:")
print(response)
````

返回结果:

```
Completion for prompt 1:
1 - 兄弟姐妹在迷人的村庄里探险时，杰克不幸滚下山丘，但两人仍然保持冒险精神。

2 - In a charming village, siblings Jack and Jill start fetching water from a well on top of a hill. Unfortunately, while climbing up with excitement, Jack trips on a rock and tumbles down the hill, with Jill following suit. Despite minor injuries, they receive comforting hugs upon returning home. Despite the accident, their adventurous spirit remains undiminished as they continue to explore happily.

3 - Jack, Jill

4 - {"french_summary": "Dans un charmant village, les frères et sœurs Jack et Jill commencent à chercher de l'eau d'un puits au sommet d'une colline. Malheureusement, alors qu'ils montent avec excitation, Jack trébuche sur un rocher et dévale la colline, suivi de Jill. Malgré des blessures mineures, ils reçoivent des câlins réconfortants en rentrant chez eux. Malgré l'accident, leur esprit d'aventure reste intact alors qu'ils continuent à explorer joyeusement.", "num_names": 2}
```

#### Tactic 1.2 Ask for output in a specified format(请求指定格式的输出)

##### 英文案例

```python
prompt_2 = f"""
Your task is to perform the following actions:
1 - Summarize the following text delimited by
  <> with 1 sentence.
2 - Translate the summary into French.
3 - List each name in the French summary.
4 - Output a json object that contains the
  following keys: french_summary, num_names.

Use the following format:
Text: <text to summarize>
Summary: <summary>
Translation: <summary translation>
Names: <list of names in Italian summary>
Output JSON: <json with summary and num_names>

Text: <{text}>
"""
response = get_completion(prompt_2)
print("\nCompletion for prompt 2:")
print(response)
```

结果:

```
Completion for prompt 2:
Summary: Jack and Jill go on a quest to fetch water, but misfortune strikes and they tumble down the hill, returning home slightly battered but with their adventurous spirits undimmed.
Translation: Jack et Jill partent en quête d'eau, mais la malchance frappe et ils dégringolent la colline, rentrant chez eux légèrement meurtris mais avec leurs esprits aventureux intacts.
Names: Jack, Jill
Output JSON: {"french_summary": "Jack et Jill partent en quête d'eau, mais la malchance frappe et ils dégringolent la colline, rentrant chez eux légèrement meurtris mais avec leurs esprits aventureux intacts.", "num_names": 2}
```

##### 中文案例

```python
text = f"""
在一个迷人的村庄里，兄弟姐妹杰克和吉尔开始从山顶的井里打水。 当他们欢呼雀跃地向上爬时，不幸降临了——杰克被一块石头绊倒，从山上滚下，吉尔也跟着跌倒。 虽然受到了轻微的打击，但两人回到家后得到了安慰的拥抱。 尽管发生了事故，他们的冒险精神丝毫未减，继续愉快地探索。
"""

prompt_2 = f"""
您的任务是执行以下操作：
1 - 用一句话总结由<>分隔的内容
2 - 将摘要翻译成英文。
3 - 在英文摘要中列出每个名字。
4 - 输出一个包含以下键：french_summary、num_names。

请你使用以下格式：
总结：<要总结的文本>
摘要：<摘要>
翻译：<摘要翻译>
姓名：<英语摘要中的姓名列表>
输出 JSON：<带有french_summary和num_names的json>

内容: <{text}>
"""
response = get_completion(prompt_2)
print("\nCompletion for prompt 2:")
print(response)



```

结果:

```
Completion for prompt 2:
总结：杰克和吉尔在山顶的井里打水时发生了意外，但他们的冒险精神未减，继续探索。
摘要：在一个迷人的村庄里，兄弟姐妹杰克和吉尔开始从山顶的井里打水。 当他们欢呼雀跃地向上爬时，不幸降临了——杰克被一块石头绊倒，从山上滚下，吉尔也跟着跌倒。 虽然受到了轻微的打击，但两人回到家后得到了安慰的拥抱。 尽管发生了事故，他们的冒险精神丝毫未减，继续愉快地探索。
翻译：In a charming village, siblings Jack and Jill started to fetch water from a well on top of a hill. Unfortunately, while they were climbing up with excitement, an accident happened - Jack tripped on a stone and rolled down the hill, with Jill falling down as well. Despite minor injuries, they received comforting hugs when they got back home. Despite the accident, their adventurous spirit remained undiminished as they continued to explore happily.
姓名：Jack、Jill
输出 JSON：{"french_summary": "Jack et Jill ont eu un accident en allant chercher de l'eau dans un puits sur une colline, mais leur esprit d'aventure est resté intact et ils ont continué à explorer joyeusement.", "num_names": 2}
```

#### Tactic 2: Instruct the model to work out its own solution before rushing to a conclusion(在匆忙下结论之前指示模型制定出自己的解决方案)

##### 英文案例

```python
prompt = f"""
Determine if the student's solution is correct or not.

Question:
I'm building a solar power installation and I need \
 help working out the financials.
- Land costs $100 / square foot
- I can buy solar panels for $250 / square foot
- I negotiated a contract for maintenance that will cost \
me a flat $100k per year, and an additional $10 / square \
foot
What is the total cost for the first year of operations
as a function of the number of square feet.

Student's Solution:
Let x be the size of the installation in square feet.
Costs:
1. Land cost: 100x
2. Solar panel cost: 250x
3. Maintenance cost: 100,000 + 100x
Total cost: 100x + 250x + 100,000 + 100x = 450x + 100,000
"""
response = get_completion(prompt)
print(response)
```

结果:

```
The student's solution is correct.
```

> Note that the student's solution is actually not correct.
> We can fix this by instructing the model to work out its own solution first.

##### 中文案例

```python
prompt = f"""
判断学生的答案是否正确。

问题：
我正在建造一个太阳能装置，我需要帮助来计算财务费用。
- 土地成本 100 美元/平方英尺
- 我可以以 250 美元/平方英尺的价格购买太阳能电池板
- 我协商了一份维护合同，每年将花费我 10 万美元，外加每平方英尺 10 美元
运营第一年的总成本与平方英尺数的关系是多少？

学生的解决方案：
设x是以平方英尺为单位的安装尺寸。
费用：
1.土地成本：100x
2.太阳能电池板成本：250x
3.维护费用：100000+100x
总成本：100x + 250x + 100,000 + 100x = 450x + 100000
"""
response = get_completion(prompt)
print(response)

```

返回结果:

```
学生的答案是正确的。
```

但是实际上是错误的

##### 英文修正案例:

````python
prompt = f"""
Your task is to determine if the student's solution \
is correct or not.
To solve the problem do the following:
- First, work out your own solution to the problem.
- Then compare your solution to the student's solution \
and evaluate if the student's solution is correct or not.
Don't decide if the student's solution is correct until
you have done the problem yourself.

Use the following format:
Question:
```
question here
```
Student's solution:
```
student's solution here
```
Actual solution:
```
steps to work out the solution and your solution here
```
Is the student's solution the same as actual solution \
just calculated:
```
yes or no
```
Student grade:
```
correct or incorrect
```

Question:
```
I'm building a solar power installation and I need help \
working out the financials.
- Land costs $100 / square foot
- I can buy solar panels for $250 / square foot
- I negotiated a contract for maintenance that will cost \
me a flat $100k per year, and an additional $10 / square \
foot
What is the total cost for the first year of operations \
as a function of the number of square feet.
```
Student's solution:
```
Let x be the size of the installation in square feet.
Costs:
1. Land cost: 100x
2. Solar panel cost: 250x
3. Maintenance cost: 100,000 + 100x
Total cost: 100x + 250x + 100,000 + 100x = 450x + 100,000
```
Actual solution:
"""
response = get_completion(prompt)
print(response)
````

结果:

```
Let x be the size of the installation in square feet.

Costs:
1. Land cost: 100x
2. Solar panel cost: 250x
3. Maintenance cost: 100,000 + 10x

Total cost: 100x + 250x + 100,000 + 10x = 360x + 100,000

Is the student's solution the same as actual solution just calculated:
No

Student grade:
Incorrect
```

##### 中文修正案例 1:

````python
prompt = f"""
您的任务是确定学生的解决方案是否正确。要解决此问题，请执行以下操作：
- 首先，找出你自己的解决方案。
- 然后将您的解决方案与学生的解决方案进行比较，并评估学生的解决方案是否正确。
在您自己完成问题之前，不要判断学生的解决方案是否正确。

使用以下格式：

问题：
```
在这里提问
```
学生的解决方案：
```
学生的解决方案在这里
```
实际解决方案：
```
在此处制定解决方案和您的解决方案的步骤
```
学生的解是否与刚刚计算的实际解决方案相同：
```
是还是不是
```
学生回答结果：
```
正确或不正确
```

问题：
我正在建造一个太阳能装置，我需要帮助来计算财务费用。
- 土地成本 100 美元/平方英尺
- 我可以以 250 美元/平方英尺的价格购买太阳能电池板
- 我协商了一份维护合同，每年将花费我10万美元，同时外加每平方英尺10美元

运营第一年的总成本与平方英尺数的关系是多少？


学生的解决方案：
```
设x是以平方英尺为单位的安装尺寸。
费用：
1.土地成本：100x
2.太阳能电池板成本：250x
3.维护费用：100000+100x
总成本：100x + 250x + 100,000 + 100x = 450x + 100000
```

"""
response = get_completion(prompt)
print(response)
````

返回结果:

```
实际解决方案：
- 运营第一年的总成本应该是土地成本、太阳能电池板成本和维护费用的总和。
- 土地成本为100美元/平方英尺，因此总土地成本为100x。
- 太阳能电池板成本为250美元/平方英尺，因此总太阳能电池板成本为250x。
- 维护费用为每平方英尺10美元，因此总维护费用为10x+100000。
- 因此，总成本为100x+250x+10x+100000=360x+100000。

学生的解是否与刚刚计算的实际解决方案相同：
是

学生回答结果：
正确
```

学生方案:`总成本：100x + 250x + 100,000 + 100x = 450x + 100000`
chatgpt 方案: `总成本为100x+250x+10x+100000=360x+100000`
结果明显不同,回答还是说学生正确

修复流程:

-   拿着结果,返回逻辑的流程,看哪一步表达导致它搞错

返回结果,发现: `学生的解是否与刚刚计算的实际解决方案相同` 这一步有问题,我们就描述详细一点
, 改成: `学生的解是否与刚刚计算的实际解决方案相同, 对比一下,如果一样就是,否则就不是：`

##### 中文修正案例 2:

````python
prompt = f"""
您的任务是确定学生的解决方案是否正确。要解决此问题，请执行以下操作：
- 首先，找出你自己的解决方案。
- 然后将您的解决方案与学生的解决方案进行比较，并评估学生的解决方案是否正确。
在您自己完成问题之前，不要判断学生的解决方案是否正确。

使用以下格式：

问题：
```
在这里提问
```
学生的解决方案：
```
学生的解决方案在这里
```
实际解决方案：
```
在此处制定解决方案和您的解决方案的步骤
```
学生的解是否与刚刚计算的实际解决方案相同, 对比一下,如果一样就是,否则就不是：
```
是还是不是
```
学生回答结果：
```
正确或不正确
```

问题：
我正在建造一个太阳能装置，我需要帮助来计算财务费用。
- 土地成本 100 美元/平方英尺
- 我可以以 250 美元/平方英尺的价格购买太阳能电池板
- 我协商了一份维护合同，每年将花费我10万美元，同时外加每平方英尺10美元

运营第一年的总成本与平方英尺数的关系是多少？


学生的解决方案：
```
设x是以平方英尺为单位的安装尺寸。
费用：
1.土地成本：100x
2.太阳能电池板成本：250x
3.维护费用：100000+100x
总成本：100x + 250x + 100,000 + 100x = 450x + 100000
```

"""
response = get_completion(prompt)
print(response)
````

返回结果:

```
实际解决方案：
- 运营第一年的总成本应该是土地成本、太阳能电池板成本和维护费用的总和。
- 土地成本为 100 美元/平方英尺，假设安装尺寸为 x 平方英尺，则土地成本为 100x 美元。
- 太阳能电池板成本为 250 美元/平方英尺，假设安装尺寸为 x 平方英尺，则太阳能电池板成本为 250x 美元。
- 维护费用为每平方英尺 10 美元，假设安装尺寸为 x 平方英尺，则维护费用为 10x 美元。另外，每年还需要支付 10 万美元的维护费用。
- 因此，运营第一年的总成本为 100x + 250x + 10x + 100000 = 360x + 100000 美元。

学生的解是否与刚刚计算的实际解决方案相同, 对比一下,如果一样就是,否则就不是：
不是

学生回答结果：
不正确
```

最后得到了我们满意的结果:

## 模型局限性：幻觉

##### 英文案例

-   Boie 是一家真实的公司，产品名称不是真实的。

```python
prompt = f"""
Tell me about AeroGlide UltraSlim Smart Toothbrush by Boie
"""
response = get_completion(prompt)
print(response)
```

返回结果

```
The AeroGlide UltraSlim Smart Toothbrush by Boie is a high-tech toothbrush that uses advanced sonic technology to provide a deep and thorough clean. It features a slim and sleek design that makes it easy to hold and maneuver, and it comes with a range of smart features that help you optimize your brushing routine.

One of the key features of the AeroGlide UltraSlim Smart Toothbrush is its advanced sonic technology, which uses high-frequency vibrations to break up plaque and bacteria on your teeth and gums. This technology is highly effective at removing even the toughest stains and buildup, leaving your teeth feeling clean and refreshed.

In addition to its sonic technology, the AeroGlide UltraSlim Smart Toothbrush also comes with a range of smart features that help you optimize your brushing routine. These include a built-in timer that ensures you brush for the recommended two minutes, as well as a pressure sensor that alerts you if you're brushing too hard.

Overall, the AeroGlide UltraSlim Smart Toothbrush by Boie is a highly advanced and effective toothbrush that is perfect for anyone looking to take their oral hygiene to the next level. With its advanced sonic technology and smart features, it provides a deep and thorough clean that leaves your teeth feeling fresh and healthy.
```

##### 中文案例

-   字节跳动没有文心一言

```python
prompt = f"""
告诉我,关于字节跳动的文心一言做了哪些创举"""
response = get_completion(prompt)
print(response)
```

返回结果:

```
字节跳动的文心一言是一款基于AI技术的智能写作工具，它可以帮助用户快速生成高质量的文章。以下是文心一言的一些创举：

1. 首创AI写作技术：文心一言是全球首款基于AI技术的智能写作工具，它可以根据用户输入的关键词和主题，自动生成高质量的文章。

2. 支持多种语言：文心一言支持多种语言，包括中文、英文、日文、韩文等，可以满足不同用户的需求。

3. 提供多种写作模板：文心一言提供多种写作模板，包括新闻报道、科技文章、商业分析等，用户可以根据自己的需求选择合适的模板。

4. 支持多种输出格式：文心一言支持多种输出格式，包括Word、PDF、HTML等，用户可以根据自己的需求选择合适的输出格式。

5. 提供多种写作辅助功能：文心一言提供多种写作辅助功能，包括关键词提取、段落优化、语法检查等，可以帮助用户提高文章质量。

总之，文心一言是一款非常实用的智能写作工具，它的出现为用户提供了一种全新的写作方式，可以帮助用户快速生成高质量的文章。
```

小结: 通过上面的案例,我们可以看到,模型的局限性,就是它只能根据你的输入,来生成一个结果,但是它并不知道这个结果是否正确,所以,我们需要自己来判断,这个结果是否正确.
