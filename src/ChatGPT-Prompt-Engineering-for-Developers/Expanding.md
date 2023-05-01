# Expanding(扩展)

本节讲的是调节模型中的`temperature`参数来获得不一样的结果, [open ai temperature](https://platform.openai.com/docs/api-reference/completions/create#completions/create-temperature) 定义:
```
What sampling temperature to use, between 0 and 2. Higher values like 0.8 will make the output more random, while lower values like 0.2 will make it more focused and deterministic.

We generally recommend altering this or `top_p` but not both.
```

解释:
> temperature 是一个0-2的一个参数, 越小确定性越强,越大随机性越大

于是教程中说, 如果你想要做一些不可预测和更具创造性的结果那么你就把temperature调大, 如果你想做一些具有确定性或预测性的那么就把temperature调小


本节实验是根据一个用户的评论来自定义回复邮件

英文评论:
```
So, they still had the 17 piece system on seasonal sale for around $49 in the month of November, about half off, but for some reason (call it price gouging) around the second week of December the prices all went up to about anywhere from between $70-$89 for the same system. And the 11 piece system went up around $10 or so in price also from the earlier sale price of $29. So it looks okay, but if you look at the base, the part where the blade locks into place doesn’t look as good as in previous editions from a few years ago, but I plan to be very gentle with it (example, I crush very hard items like beans, ice, rice, etc. in the blender first then pulverize them in the serving size I want in the blender then switch to the whipping blade for a finer flour, and use the cross cutting blade first when making smoothies, then use the flat blade if I need them finer/less pulpy). Special tip when making smoothies, finely cut and freeze the fruits and vegetables (if using spinach-lightly stew soften the spinach then freeze until ready for use-and if making sorbet, use a small to medium sized food processor) that you plan to use that way you can avoid adding so much ice if at all-when making your smoothie. After about a year, the motor was making a funny noise. I called customer service but the warranty expired already, so I had to buy another one. FYI: The overall quality has gone done in these types of products, so they are kind of counting on brand recognition and consumer loyalty to maintain sales. Got it in about two days.
```
翻译:
```
因此，他们在 11 月份仍然以 49 美元左右的价格对 17 件系统进行季节性销售，大约减半，但由于某种原因（称之为价格欺诈），在 12 月的第二周左右，价格全部上涨至大约任何地方 同一系统在 70-89 美元之间。 11 件套系统的价格也比之前的 29 美元上涨了 10 美元左右。 所以它看起来还不错，但是如果你看一下底座，刀片锁定到位的部分看起来不像几年前的以前版本那么好，但我打算对它非常温和（例如，我 先在搅拌机中粉碎非常硬的东西，如豆子、冰、大米等，然后在搅拌机中将它们粉碎成我想要的份量，然后切换到搅打刀片以获得更细的面粉，并在制作冰沙时先使用横切刀片 ，然后如果我需要它们更细/更少浆状，请使用平刀片）。 制作冰沙时的特别提示，切碎并冷冻您计划使用的水果和蔬菜（如果使用菠菜 - 稍微炖软菠菜，然后冷冻直至准备使用 - 如果制作冰糕，请使用中小型食品加工机） 这样你就可以避免在制作冰沙时添加太多冰块。 大约一年后，电机发出一种奇怪的声音。 我打电话给客户服务，但保修期已经过期，所以我不得不再买一个。 仅供参考：这些类型的产品的整体质量已经过时，因此他们有点依赖品牌知名度和消费者忠诚度来维持销售。 大概两天就拿到了
```

## Customize the automated reply to a customer email(自定义对客户电子邮件的自动回复)

````python
import openai
import os

from dotenv import load_dotenv, find_dotenv
_ = load_dotenv(find_dotenv()) # read local .env file

openai.api_key  = os.getenv('OPENAI_API_KEY')

def get_completion(prompt, model="gpt-3.5-turbo",temperature=0): # Andrew mentioned that the prompt/ completion paradigm is preferable for this class
    messages = [{"role": "user", "content": prompt}]
    response = openai.ChatCompletion.create(
        model=model,
        messages=messages,
        temperature=temperature, # this is the degree of randomness of the model's output
    )
    return response.choices[0].message["content"]
# given the sentiment from the lesson on "inferring",
# and the original customer message, customize the email
sentiment = "negative"

# review for a blender
review = f"""
So, they still had the 17 piece system on seasonal \
sale for around $49 in the month of November, about \
half off, but for some reason (call it price gouging) \
around the second week of December the prices all went \
up to about anywhere from between $70-$89 for the same \
system. And the 11 piece system went up around $10 or \
so in price also from the earlier sale price of $29. \
So it looks okay, but if you look at the base, the part \
where the blade locks into place doesn’t look as good \
as in previous editions from a few years ago, but I \
plan to be very gentle with it (example, I crush \
very hard items like beans, ice, rice, etc. in the \ 
blender first then pulverize them in the serving size \
I want in the blender then switch to the whipping \
blade for a finer flour, and use the cross cutting blade \
first when making smoothies, then use the flat blade \
if I need them finer/less pulpy). Special tip when making \
smoothies, finely cut and freeze the fruits and \
vegetables (if using spinach-lightly stew soften the \ 
spinach then freeze until ready for use-and if making \
sorbet, use a small to medium sized food processor) \ 
that you plan to use that way you can avoid adding so \
much ice if at all-when making your smoothie. \
After about a year, the motor was making a funny noise. \
I called customer service but the warranty expired \
already, so I had to buy another one. FYI: The overall \
quality has gone done in these types of products, so \
they are kind of counting on brand recognition and \
consumer loyalty to maintain sales. Got it in about \
two days.
"""

prompt = f"""
You are a customer service AI assistant.
Your task is to send an email reply to a valued customer.
Given the customer email delimited by ```, \
Generate a reply to thank the customer for their review.
If the sentiment is positive or neutral, thank them for \
their review.
If the sentiment is negative, apologize and suggest that \
they can reach out to customer service. 
Make sure to use specific details from the review.
Write in a concise and professional tone.
Sign the email as `AI customer agent`.
Customer review: ```{review}```
Review sentiment: {sentiment}
"""
response = get_completion(prompt)
print(response)
````
返回结果:
```
Dear Valued Customer,

Thank you for taking the time to leave a review about our product. We are sorry to hear that you experienced an increase in price and that the quality of the product did not meet your expectations. We apologize for any inconvenience this may have caused you.

We would like to assure you that we take all feedback seriously and we will be sure to pass your comments along to our team. If you have any further concerns, please do not hesitate to reach out to our customer service team for assistance.

Thank you again for your review and for choosing our product. We hope to have the opportunity to serve you better in the future.

Best regards,

AI customer agent
```
翻译:
```
尊贵的顾客，

感谢您抽出宝贵时间对我们的产品发表评论。 我们很遗憾听到您遇到价格上涨并且产品质量没有达到您的期望。 对于由此给您带来的任何不便，我们深表歉意。

我们向您保证，我们会认真对待所有反馈，并且一定会将您的意见转达给我们的团队。 如果您有任何其他疑虑，请随时联系我们的客户服务团队寻求帮助。

再次感谢您的评论和选择我们的产品。 我们希望将来有机会为您提供更好的服务。

Best regards,

AI customer agent
```


多次运行,发现返回的结果都是一样, 因为我们在上面的参数设置里面设置了: `temperature=0`, 所以不管运行多少次,结果都是确定性的


## Remind the model to use details from the customer's email


现在我们把`temperature=0.7`, 或者你设置其他参数来使每次运行的结果不一样:

````python
prompt = f"""
You are a customer service AI assistant.
Your task is to send an email reply to a valued customer.
Given the customer email delimited by ```, \
Generate a reply to thank the customer for their review.
If the sentiment is positive or neutral, thank them for \
their review.
If the sentiment is negative, apologize and suggest that \
they can reach out to customer service. 
Make sure to use specific details from the review.
Write in a concise and professional tone.
Sign the email as `AI customer agent`.
Customer review: ```{review}```
Review sentiment: {sentiment}
"""
response = get_completion(prompt, temperature=0.7)
print(response)
````

第一次运行结果:
```
Dear Valued Customer,

Thank you for taking the time to share your experience with our product. We apologize for any inconvenience you may have experienced regarding the price changes and the quality of our product. 

We understand your frustration and we would like to assist you further. Please do not hesitate to contact our customer service team for further assistance. We value your feedback and we will make sure that we take your concerns into consideration in order to improve our services.

Thank you again for your feedback and for choosing our product. 

Sincerely, 

AI customer agent
```

第二次运行结果:
```
Dear Valued Customer,

We are sorry to hear that your experience with our product did not meet your expectations. We apologize for any inconvenience you may have experienced due to the unexpected price increase. 

We appreciate your detailed feedback on the product, and we take your concerns seriously. We encourage you to reach out to our customer service team, who can help address any further issues you may have encountered with the motor.

Thank you for taking the time to share your thoughts with us, as we are always looking for ways to improve our products and services. 

Sincerely,

AI customer agent
```

你会发现每次运行出来的结果是不一样的

## 本节小结

本节主要是讲的temperature参数,它是一个0-2的一个参数, 越小确定性越强,越大随性越大,我们可以更加我们的场景来定义该参数以获得我们想要的效果









