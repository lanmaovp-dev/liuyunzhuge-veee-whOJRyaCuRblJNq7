[合集 \- 机器学习(6\)](https://github.com)[1\.从零开始学机器学习——什么是机器学习09\-24](https://github.com/guoxiaoyu/p/18412875)[2\.从零开始学机器学习——了解回归09\-25](https://github.com/guoxiaoyu/p/18413894)[3\.从零开始学机器学习——准备和可视化数据09\-27](https://github.com/guoxiaoyu/p/18419035):[milou加速器](https://xinminxuehui.org)[4\.从零开始学机器学习——线性和多项式回归09\-29](https://github.com/guoxiaoyu/p/18421693)[5\.从零开始学机器学习——逻辑回归09\-30](https://github.com/guoxiaoyu/p/18429831)6\.从零开始学机器学习——网络应用10\-06收起
首先给大家介绍一个很好用的学习地址：[https://cloudstudio.net/columns](https://github.com)


今天，我们的主要任务是按照既定的流程再次运行模型，并将其成功加载到 Web 应用程序中，以便通过 Web 界面进行调用。最终生成的模型将能够基于 UFO 目击事件的数据和经纬度信息，推断出事件发生的城市地址。尽管经纬度信息似乎已经足够，但我们还是需要模型进行预测，这只是一个练习目的。希望通过这个过程，你能进一步理解机器学习模型在 Web 应用中的应用与整合。


# 知识回顾


## 工具


对于此任务，你需要两个工具：Flask 和 Pickle，它们都在 Python 上运行。


**Flask** 是一个轻量级的 web 应用框架，适用于构建简单的 web 应用和 RESTful API。它遵循 WSGI（Web Server Gateway Interface）标准，支持多种扩展，具有以下特点：


* **简单易用**：Flask 的核心非常简单，易于上手，适合初学者。
* **灵活性**：开发者可以根据需要选择和添加扩展，使其具有更强的功能。
* **路由系统**：Flask 提供了灵活的 URL 路由机制，方便管理不同的页面和请求。
* **模板引擎**：集成了 Jinja2 模板引擎，可以轻松生成动态 HTML 页面。
* **支持 RESTful API**：非常适合构建 RESTful 服务。


**Pickle** 是 Python 的一个内置模块，用于对象序列化和反序列化。序列化是将 Python 对象转换为字节流的过程，而反序列化则是将字节流还原为 Python 对象。它的特点包括：


* **简单方便**：使用简单，可以轻松地保存和加载 Python 对象。
* **支持多种对象**：可以序列化大多数 Python 数据类型，包括自定义类。
* **持久化数据**：适合将数据存储到文件中，以便后续使用。


**注意**：Pickle 对安全性有一定的隐患，因为反序列化不可信的数据可能导致代码执行漏洞。


## 清洗数据


首先，让我们来深入了解一下我们的数据集，看看它的具体结构和内容。这一步非常重要，因为数据的质量和特征将直接影响到模型的性能和预测结果。



```
import pandas as pd
import numpy as np

ufos = pd.read_csv('./data/ufos.csv')
ufos.head()

```

![image](https://img2024.cnblogs.com/blog/1423484/202409/1423484-20240928172252565-150597068.png)


因此，根据我们要解决的问题，我们需要提取与之相关的向量字段数据。这些字段包括：城市名称、经纬度坐标以及目击时长。这些信息将为我们后续的分析和建模提供重要的输入特征，帮助我们准确地推断 UFO 目击事件发生的城市。



```
ufos = pd.DataFrame({'Seconds': ufos['duration (seconds)'], 'Country': ufos['country'],'Latitude': ufos['latitude'],'Longitude': ufos['longitude']})

ufos.Country.unique()
ufos.dropna(inplace=True)
ufos = ufos[(ufos['Seconds'] >= 1) & (ufos['Seconds'] <= 60)]
ufos.info()

```

在完成这一系列的数据清洗和筛选后，我们的数据集将仅包含我们所需的相关字段，确保其精简而高效。此外，我们还对目击时长进行了特别的过滤，以确保该特征的有效性和一致性。


因为我们了解到，模型对数据的敏感性会影响其预测结果。接下来，我们需要将文本字段转换为适合模型处理的数值数据格式。这一转换过程将使得模型能够理解和利用这些文本信息，从而提升预测能力。



```
from sklearn.preprocessing import LabelEncoder
ufos['Country'] = LabelEncoder().fit_transform(ufos['Country'])
ufos.head()

```

目前为止，我们已经成功完成了数据的清理和预处理，确保我们的数据集具备良好的质量和一致性。接下来，我们将进入模型训练的阶段。考虑到我们的任务是预测城市，而城市名称是一个有限且固定的类别，逻辑回归模型将是一个理想的选择。


## 建立模型


现在，我们将开始准备训练模型的关键步骤，即将数据集划分为训练组和测试组。



```
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, classification_report 
from sklearn.linear_model import LogisticRegression

Selected_features = ['Seconds','Latitude','Longitude']

X = ufos[Selected_features]
y = ufos['Country']

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=0)


model = LogisticRegression()
model.fit(X_train, y_train)
predictions = model.predict(X_test)

print(classification_report(y_test, predictions))
print('Predicted labels: ', predictions)
print('Accuracy: ', accuracy_score(y_test, predictions))

```

返回如下：



```
              precision    recall  f1-score   support

           0       1.00      1.00      1.00        41
           1       0.83      0.23      0.36       250
           2       1.00      1.00      1.00         8
           3       1.00      1.00      1.00       131
           4       0.96      1.00      0.98      4743

    accuracy                           0.96      5173
   macro avg       0.96      0.85      0.87      5173
weighted avg       0.96      0.96      0.95      5173

Predicted labels:  [4 4 4 ... 3 4 4]
Accuracy:  0.9605644693601392

```

模型的准确率相当不错，约为95%。这一结果并不令人意外，因为我们的特征——国家（Country）以及经纬度（Latitude/Longitude）信息之间确实存在显著的相关性。


接下来，我们需要将这一训练好的模型进行打包，以便将其集成到我们的Web应用程序中。这样，用户就能够通过网页轻松调用模型进行城市预测，享受流畅的交互体验。


## 打包模型


我们可以直接利用pickle库这一便捷的依赖，将训练好的模型序列化并写入文件。



```
import pickle
model_filename = 'ufo-model.pkl'
pickle.dump(model, open(model_filename,'wb'))

model = pickle.load(open('ufo-model.pkl','rb'))

```

# 构建WEB


现在，我们的目标是通过Flask应用程序来调用已训练好的模型，并将预测结果以更加美观和用户友好的方式展示在Web页面上。Flask作为一个轻量级的Web框架，能够帮助我们快速构建Web应用，让用户通过直观的界面与模型进行交互。



```
web-app/
  static/
    css/
  templates/
notebook.ipynb
ufo-model.pkl
app.py
requirements.txt

```

这是我们需要新建的目录结构，请务必严格遵循这一布局，因为不按此结构组织文件和文件夹可能会导致程序在运行时出现错误。


requirements.txt中添加以下几行：



```
scikit-learn
pandas
numpy
flask

```

然后安装相关依赖



> pip install \-r requirements.txt


其余的CSS和HTML代码请前往学习中心自行复制和参考，这里不再一一列举。


在app.py中添加:



```
import numpy as np
from flask import Flask, request, render_template
import pickle

app = Flask(__name__)

model = pickle.load(open("./ufo-model.pkl", "rb"))


@app.route("/")
def home():
    return render_template("index.html")


@app.route("/predict", methods=["POST"])
def predict():

    int_features = [int(x) for x in request.form.values()]
    final_features = [np.array(int_features)]
    prediction = model.predict(final_features)

    output = prediction[0]

    countries = ["Australia", "Canada", "Germany", "UK", "US"]

    return render_template(
        "index.html", prediction_text="Likely country: {}".format(countries[output])
    )


if __name__ == "__main__":
    app.run(debug=True)

```

这段代码实现了一个简单的 web 应用，允许用户输入特征数据并根据加载的机器学习模型进行预测。预测结果会被渲染到同一个页面上显示给用户。通过 Flask 的路由系统，可以方便地处理用户请求和响应。


例如，当我们输入特征数据“50, 44, \-12”时，应用将调用训练好的模型进行计算，并在页面上展示预测结果，如图所示。


![image](https://img2024.cnblogs.com/blog/1423484/202409/1423484-20240928172308831-859359704.png)


# 总结


在这个项目中，我们通过使用 Flask 和 Pickle 将一个机器学习模型成功集成到 Web 应用中，使用户能够通过友好的界面进行预测。这一过程不仅让我们体验到了模型训练与数据预处理的细节，也深刻理解了如何在实际应用中实现机器学习的功能。


通过这一系列的实践操作，我们不仅巩固了对工具和技术的理解，也提升了将理论知识转化为实际应用的能力。这一过程的重要性在于，它不仅是对我们技术能力的挑战，更是对解决实际问题能力的培养。




---


我是努力的小雨，一名 Java 服务端码农，潜心研究着 AI 技术的奥秘。我热爱技术交流与分享，对开源社区充满热情。同时也是一位腾讯云创作之星、阿里云专家博主、华为云云享专家、掘金优秀作者。


💡 我将不吝分享我在技术道路上的个人探索与经验，希望能为你的学习与成长带来一些启发与帮助。


🌟 欢迎关注努力的小雨！🌟


