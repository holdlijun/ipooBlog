---
title: "Learn Md"
date: 2020-03-18T11:06:34+08:00
draft: false
tags : ["Markdown"]
---

#  标题

| Markdown | HTML | 
| ------ | ------ | 
| # Heading level 1	 | 	一级
| ## Heading level 2	| 二级
| ### Heading level 3	| 	三级
| #### Heading level 4	 | 	四级
| ##### Heading level 5	 | 	五级
| ###### Heading level 6	 | 	六级

# 段落

创建段落，使用一行或多行分割。

不要缩进带有空格或制表符的段落。 <br> 

# 加粗文本

请在单词或短语的前后添加两个星号或下划线。要加粗一个单词的中部以强调，请在字母周围添加两个星号，且各空格之间没有空格。  

	public void consumer() {
        Consumer consumer = createConsumer();
        consumer.subscribe(RocketMqConfig.topicJudge, "*", (message, context) -> {
            String body = new String(message.getBody());
            try {
                if (message.getTag().equals(RocketMqConfig.RISK_TAG) || message.getTag().equals(RocketMqConfig.WAIT_TAG)) {
                    riskProcess.riskRule(body);
                }
            } catch (Throwable e) {
                e.printStackTrace();
            }
            return Action.CommitMessage;
        });
        consumer.start();
        logger.warn("启动一般消费者正常");
    }

:smile:
