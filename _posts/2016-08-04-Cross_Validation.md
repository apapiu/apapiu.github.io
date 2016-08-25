
---
layout: post
title: Tf-Idf Ridge Model Selection using Pipelines in Sklearn
---


### Creating a pipeline to tune  tf-idf + ridge regularization parameters and select the best model.



```python
import pandas as pd
import numpy as np

from sklearn import linear_model
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.pipeline import Pipeline
from sklearn.grid_search import GridSearchCV


from matplotlib import pyplot as plt
% matplotlib inline
```

We are going to try to build a model that predicts the salary offer for a job based on the description of the job listing.


```python
train = pd.read_csv("https://raw.githubusercontent.com/ajschumacher/gadsdata/master/salary/train.csv")
```


```python
y = train.SalaryNormalized
```


```python
train.head(3)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Id</th>
      <th>Title</th>
      <th>FullDescription</th>
      <th>LocationRaw</th>
      <th>LocationNormalized</th>
      <th>ContractType</th>
      <th>ContractTime</th>
      <th>Company</th>
      <th>Category</th>
      <th>SalaryRaw</th>
      <th>SalaryNormalized</th>
      <th>SourceName</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>12612628</td>
      <td>Engineering Systems Analyst</td>
      <td>Engineering Systems Analyst Dorking Surrey Sal...</td>
      <td>Dorking, Surrey, Surrey</td>
      <td>Dorking</td>
      <td>NaN</td>
      <td>permanent</td>
      <td>Gregory Martin International</td>
      <td>Engineering Jobs</td>
      <td>20000 - 30000/annum 20-30K</td>
      <td>25000</td>
      <td>cv-library.co.uk</td>
    </tr>
    <tr>
      <th>1</th>
      <td>12612830</td>
      <td>Stress Engineer Glasgow</td>
      <td>Stress Engineer Glasgow Salary **** to **** We...</td>
      <td>Glasgow, Scotland, Scotland</td>
      <td>Glasgow</td>
      <td>NaN</td>
      <td>permanent</td>
      <td>Gregory Martin International</td>
      <td>Engineering Jobs</td>
      <td>25000 - 35000/annum 25-35K</td>
      <td>30000</td>
      <td>cv-library.co.uk</td>
    </tr>
    <tr>
      <th>2</th>
      <td>12612844</td>
      <td>Modelling and simulation analyst</td>
      <td>Mathematical Modeller / Simulation Analyst / O...</td>
      <td>Hampshire, South East, South East</td>
      <td>Hampshire</td>
      <td>NaN</td>
      <td>permanent</td>
      <td>Gregory Martin International</td>
      <td>Engineering Jobs</td>
      <td>20000 - 40000/annum 20-40K</td>
      <td>30000</td>
      <td>cv-library.co.uk</td>
    </tr>
  </tbody>
</table>
</div>



We will just use the description and build a pipeline - this is insanely easy in sklearn!


```python
estimators = [("tf_idf", TfidfVectorizer()), 
              ("ridge", linear_model.Ridge())]
model = Pipeline(estimators)
```

#### So we just plug in the raw descriptions and the tf_idf transforms it into a matrix that is then fitted by the ridge model. Genius. 

 ### $$Description\longrightarrow X , y \longrightarrow model$$


```python
model.fit(train.FullDescription, y) 
```




    Pipeline(steps=[('tf_idf', TfidfVectorizer(analyzer='word', binary=False, decode_error='strict',
            dtype=<class 'numpy.int64'>, encoding='utf-8', input='content',
            lowercase=True, max_df=1.0, max_features=None, min_df=1,
            ngram_range=(1, 1), norm='l2', preprocessor=None, smooth_idf=True,
    ...it_intercept=True, max_iter=None,
       normalize=False, random_state=None, solver='auto', tol=0.001))])




```python
params = {"ridge__alpha":[0.1, 0.3, 1, 3, 10], #regularization param
          "tf_idf__min_df": [1, 3, 10], #min count of words allowed
          "tf_idf__ngram_range": [(1,1), (1,2)], #1-grams or 2-grams
          "tf_idf__stop_words": [None, "english"]} #use stopwords or don't
```

How many different model must we run? Well since we're doing a grid search we can just multiply the possibilities for each parameter to get `5*3*2*2` for a total of 60 models - a decent number. And keep in mind that for each model we have to build the tf_idf vectorizer all over again.


```python
grid = GridSearchCV(estimator=model, param_grid = params, scoring = "mean_squared_error")
```

grid.fit(train.FullDescription, y)


```python
grid.best_params_
```




    {'ridge__alpha': 0.3,
     'tf_idf__min_df': 1,
     'tf_idf__ngram_range': (1, 2),
     'tf_idf__stop_words': 'english'}




```python
np.sqrt(-grid.best_score_)
```




    10532.473521325306



We can also look at all the params:


```python
params = pd.DataFrame([i[0] for i in grid.grid_scores_])
results = pd.DataFrame(grid.grid_scores_)
results = pd.concat([params, results], 1)
results["rmse"] = np.sqrt(-results.mean_validation_score)
```


```python
results.head(3)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ridge__alpha</th>
      <th>tf_idf__min_df</th>
      <th>tf_idf__ngram_range</th>
      <th>tf_idf__stop_words</th>
      <th>parameters</th>
      <th>mean_validation_score</th>
      <th>cv_validation_scores</th>
      <th>rmse</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.1</td>
      <td>1</td>
      <td>(1, 1)</td>
      <td>None</td>
      <td>{'ridge__alpha': 0.1, 'tf_idf__stop_words': No...</td>
      <td>-1.383986e+08</td>
      <td>[-103831685.851, -141229157.862, -170145315.841]</td>
      <td>11764.293270</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.1</td>
      <td>1</td>
      <td>(1, 1)</td>
      <td>english</td>
      <td>{'ridge__alpha': 0.1, 'tf_idf__stop_words': 'e...</td>
      <td>-1.408870e+08</td>
      <td>[-105929048.004, -144749023.148, -171993435.294]</td>
      <td>11869.583228</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.1</td>
      <td>1</td>
      <td>(1, 2)</td>
      <td>None</td>
      <td>{'ridge__alpha': 0.1, 'tf_idf__stop_words': No...</td>
      <td>-1.113026e+08</td>
      <td>[-77620035.3972, -108499379.09, -147798481.11]</td>
      <td>10550.004578</td>
    </tr>
  </tbody>
</table>
</div>



### Examining the Best Model:


```python
model = grid.best_estimator_
```

Every time we predict the model will run the tf-idf part first, already fitted on the train set and then use the ridge regression model. 



```python
model.predict(train.FullDescription)
```




    array([ 25975.84531928,  32824.5058169 ,  32127.26976225, ...,
            50386.2916183 ,  50138.40072399,  27588.69246637])



One issue with using the pipeline is that we don't see the little details that go into fitting the models.

What if we want to examing more closely what goes on in each model? Say for example I want to look at the coefficients of my linear regression. That's also pretty straighforward using the `named_steps` method.


```python
grid.best_estimator_.named_steps["ridge"].coef_
```




    array([ -465.8824938 ,  1697.39286267,  1304.56896049, ...,  1416.89223231,
            -596.29992468,  -596.29992468])




```python
ridge_model = model.named_steps["ridge"]
tf_idf_model = model.named_steps["tf_idf"]
```


```python
coefficients = pd.DataFrame({"names":tf_idf_model.get_feature_names(),
                             "coef":ridge_model.coef_})
```

Let's look at the tokens with the largest coefficients:


```python
coefficients.sort_values("coef", ascending=False).head(10)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>coef</th>
      <th>names</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>88432</th>
      <td>51890.453609</td>
      <td>consultant grade</td>
    </tr>
    <tr>
      <th>235331</th>
      <td>48488.999766</td>
      <td>locum</td>
    </tr>
    <tr>
      <th>399929</th>
      <td>45052.453063</td>
      <td>subsea</td>
    </tr>
    <tr>
      <th>174963</th>
      <td>43441.208259</td>
      <td>global</td>
    </tr>
    <tr>
      <th>235338</th>
      <td>40651.232641</td>
      <td>locum consultant</td>
    </tr>
    <tr>
      <th>90843</th>
      <td>40016.083870</td>
      <td>contract</td>
    </tr>
    <tr>
      <th>235682</th>
      <td>38554.092136</td>
      <td>london</td>
    </tr>
    <tr>
      <th>211090</th>
      <td>36076.259999</td>
      <td>investment</td>
    </tr>
    <tr>
      <th>121657</th>
      <td>34280.854922</td>
      <td>director</td>
    </tr>
    <tr>
      <th>244094</th>
      <td>33309.500667</td>
      <td>manager</td>
    </tr>
  </tbody>
</table>
</div>



We see some of the usual suspects - such as london, consultant, director and manager. However given how many features we have (over 400k) it's hard to interpret these coefficients very accurately. Perhaps doing a Lasso model with a strong $l_1$ regularization might help with that, since that wil reduce the number of non-zero coefficients.


```python

```
