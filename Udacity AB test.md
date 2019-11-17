
Undacity A/B Test Design 
<br>
*Erjun Qiu*

# Experiment Overview: Free Trial Screener

At the time of this experiment, Udacity courses currently have two options on the course overview page: "start free trial", and "access course materials". If the student clicks "start free trial", they will be asked to enter their credit card information, and then they will be enrolled in a free trial for the paid version of the course. After 14 days, they will automatically be charged unless they cancel first. If the student clicks "access course materials", they will be able to view the videos and take the quizzes for free, but they will not receive coaching support or a verified certificate, and they will not submit their final project for feedback.

In the experiment, Udacity tested a change where if the student clicked "start free trial", they were asked how much time they had available to devote to the course. If the student indicated 5 or more hours per week, they would be taken through the checkout process as usual. If they indicated fewer than 5 hours per week, a message would appear indicating that Udacity courses usually require a greater time commitment for successful completion, and suggesting that the student might like to access the course materials for free. At this point, the student would have the option to continue enrolling in the free trial, or access the course materials for free instead.The following screenshot shows what the experiment looks like.


```python
from IPython.display import Image
Image(filename='/Users/erjunqiu/Desktop/Udacity AB test/Final Project_ Experiment Screenshot.png') 
```




![png](output_4_0.png)



The hypothesis was that this might set clearer expectations for students upfront, thus reducing the number of frustrated students who left the free trial because they didn't have enough time—without significantly reducing the number of students to continue past the free trial and eventually complete the course. If this hypothesis held true, Udacity could improve the overall student experience and improve coaches' capacity to support students who are likely to complete the course.

The unit of diversion is a cookie, although if the student enrolls in the free trial, they are tracked by user-id from that point forward. The same user-id cannot enroll in the free trial twice. For users that do not enroll, their user-id is not tracked in the experiment, even if they were signed in when they visited the course overview page.

# A/B TEST DESIGN AND IMPLEMENT

I use Google's [Goals-Signals-Metrics](https://library.gv.com/how-to-choose-the-right-ux-metrics-for-your-product-5f46359ab5be) process to help me build intuition to set up metrics.

## 1.Goal

Evaluate if the pop-up screener set clearer expectations for students upfront, thus **reducing the number of frustrated students who left the free trial because they didn't have enough time—without significantly reducing the number of students to continue past the free trial and eventually complete the course**.

## 2.Signal

Users might change their choice after they see the pop-up screen.

The actions after pop-up might change in two ways:
    - continue enrolling in the free trial
    - access the course materials for free

## 3.Metric Choice

Which of the following metrics would you choose to measure for this experiment and why? For each metric you choose, indicate whether you would use it as an invariant metric or an evaluation metric. The practical significance boundary for each metric, that is, the difference that would have to be observed before that was a meaningful change for the business, is given in parentheses. All practical significance boundaries are given as absolute changes.

Any place "unique cookies" are mentioned, the uniqueness is determined by day. (That is, the same cookie visiting on different days would be counted twice.) User-ids are automatically unique since the site does not allow the same user-id to enroll twice.

**Number of cookies:** That is, number of unique cookies to view the course overview page. (dmin=3000)
<br>
**Number of user-ids:** That is, number of users who enroll in the free trial. (dmin=50)
<br>
**Number of clicks:** That is, number of unique cookies to click the "Start free trial" button (which happens before the free trial screener is trigger). (dmin=240)
<br>
**Click-through-probability:** That is, number of unique cookies to click the "Start free trial" button divided by number of unique cookies to view the course overview page. (dmin=0.01)
<br>
**Gross conversion:** That is, number of user-ids to complete checkout and enroll in the free trial divided by number of unique cookies to click the "Start free trial" button. (dmin= 0.01)
<br>
**Retention:** That is, number of user-ids to remain enrolled past the 14-day boundary (and thus make at least one payment) divided by number of user-ids to complete checkout. (dmin=0.01)
<br>
**Net conversion:** That is, number of user-ids to remain enrolled past the 14-day boundary (and thus make at least one payment) divided by the number of unique cookies to click the "Start free trial" button. (dmin= 0.0075)

---

**Invariant Metrics:**
    <br>
    Number of cookies - Since we assigned cookies (unit of diversion) to control and experiment groups randomly, it's good invariant population size.
    <br>
    Number of clicks - The "click" action happens before the pop-up screen, so the users' activities stay as usual.
    <br>
    Click-through-probability - This is a ratio of the number of clicks given the number of cookies.
    
**Evaluation Metrics:**
    <br>
    Gross conversion - This is a ratio of the number of enrolled unique user-ids given the number of cookies. Because the enroll action happens after the pop-up screen, it might change.
    <br>
    Retention -  This a ratio of the number of paid unique user-ids given the number of enrolled unique user-ids. Since both the numerator and denominator might change after the pop-up screen, this metric might change.
    <br>
    Net conversion - This is a ratio of the number of paid unique user-ids given the number of cookies. Because the pay action happens after the pop-up screen, it might change.
    
**Unselected Metrics:**
    <br>
    Number of user-ids - Notice the enrolled action happens after the pop-up screen, it might change. I prefer to consider it as a change metric that lacks usability rather than directly negating it is an evaluation metric. The reason is that this metric does not have normalized. 

If pass the sanity check, we can start working on the evaluation process.

To launch the new feature, I expect the experiment results will follow the hypothesis standard:
    - reducing the number of frustrated students who left the free trial because they didn't have enough time
    - without significantly reducing the number of students to continue past the free trial and eventually complete the course
    
**Simply speaking, we need to reduce the number of enrolled users leaving the free trial and maintain remain enrolled users.**
 
Expected change from metrics:
    - Gross conversion decreases significantly
    - Retention increases significantly
    - Net conversion does not decrease significantly

## 4.Measuring Variability

This [spreadsheet](https://docs.google.com/spreadsheets/d/1MYNUtC47Pg8hdoCjOXaHqF-thheGpUshrFA21BAJnNc/edit#gid=0) contains rough estimates of the baseline values for these metrics (again, these numbers have been changed from Udacity's true numbers).

For each metric you selected as an evaluation metric, estimate its standard deviation analytically, given sample size of 5000 cookies visiting the course overview page.

---


```python
from tabulate import tabulate

print(tabulate([['Unique cookies to view course overview page per day',40000,'Number of cookies'],
               ['Unique cookies to click "Start free trial" per day',3200,'Number of user-ids'],
               ['Enrollments per day',660,'Number of clicks'],
               ['Click-through-probability on "Start free trial"',.08,'Click-through-probability'],
               ['Probability of enrolling, given click',.20625,'Gross conversion'],
               ['Probability of payment, given enroll',.53,'Retention'],
               ['Probability of payment, given click',.1093125,'Net conversion']],
               tablefmt='orgtbl',
               headers=['Description','Baseline Value','Metrics']))
```

    | Description                                         |   Baseline Value | Metrics                   |
    |-----------------------------------------------------+------------------+---------------------------|
    | Unique cookies to view course overview page per day |     40000        | Number of cookies         |
    | Unique cookies to click "Start free trial" per day  |      3200        | Number of user-ids        |
    | Enrollments per day                                 |       660        | Number of clicks          |
    | Click-through-probability on "Start free trial"     |         0.08     | Click-through-probability |
    | Probability of enrolling, given click               |         0.20625  | Gross conversion          |
    | Probability of payment, given enroll                |         0.53     | Retention                 |
    | Probability of payment, given click                 |         0.109313 | Net conversion            |


Before calculating the variability we should understand the units of analysis where the metrics based on.

In this case, Gross conversion and Net conversion based on clicks (cookies); Retention based on enrollment (user-ids).
<br>
**clicks based:**
    n_clicks = Click-through-probability * 5000 = 400
<br>
**enroll based:**
    n_enroll = Gross converion * 400 = 82.5


```python
import math

print(tabulate([['Gross conversion',math.sqrt(.20625*(1-.20625)/400)],
               ['Retention',math.sqrt(.53*(1-.53)/82.5)],
               ['Net conversion ',math.sqrt(.109313*(1-.109313)/400)]],
               tablefmt='orgtbl',
               headers=['Metrics','Standard Error']))
```

    | Metrics          |   Standard Error |
    |------------------+------------------|
    | Gross conversion |        0.0202306 |
    | Retention        |        0.054949  |
    | Net conversion   |        0.0156016 |


#### Comparison between analytical and empirical measures

Although we calculated the standard error above analytically, it doesn't mean the result is accurate. Take a look at the metrics carefully, if the unit of analysis (the metric's denominator) and the unit of diversion (the diversion used to implement experiment) are different, the variability will be different as well. As we see, the retention has higher variability than that of gross conversion and net conversion.

The reason why the variability between the unit of analysis (user-ids) and the unit of diversion (cookies) make such a difference in variability is that when calculating variability analytically, we make assumptions not only the distribution of the data but also the independence of the unit. The key point is that if the unit of diversion and unit of analysis are different, that's mean the units of diversion correlate with each other, then the analytical variability will underestimate.

In this case, the gross conversion and net conversion both have the same unit of analysis and diversion (cookies), using the analytical way to calculate variability is proper. It turns out that the variability of retention is biased because of the difference between the unit of analysis (user-ids) and the unit of diversion (cookies).

Practically, I expect to calculate the variability of gross conversion and net conversion analytically; calculate the variability of retention empirically.

## 5.Sizing

### Choosing Number of Samples given Power

Using the analytic estimates of variance, how many pageviews total (across both groups) would you need to collect to adequately power the experiment? Use an alpha of 0.05 and a beta of 0.2. Make sure you have enough power for each metric.

---

You can use the [online calculator](http://www.evanmiller.org/ab-testing/sample-size.html) or use statistical packages to calculate the sample size you need.


```python
print(tabulate([['Gross conversion','click',25835],
               ['Retention','user-id(enrolled)',39115],
               ['Net conversion','click',27413]],
               tablefmt='orgtbl',
               headers=['Metrics','Unit of Analysis','Sample size']))
```

    | Metrics          | Unit of Analysis   |   Sample size |
    |------------------+--------------------+---------------|
    | Gross conversion | click              |         25835 |
    | Retention        | user-id(enrolled)  |         39115 |
    | Net conversion   | click              |         27413 |



```python
import pandas as pd
pd.set_option('display.float_format',lambda x:'%.5f'%x)

print(tabulate([['Gross conversion','click',25835,math.ceil(25835/.08*2)],
                ['Retention','user-id(enrolled)',39115,math.ceil(39115/(660/40000)*2)],
                ['Net conversion','click',27413,math.ceil(27413/.08*2)]],
               tablefmt='orgtbl',
               headers=['Metrics','Unit of Analysis','Sample size','Pageviews']))
```

    | Metrics          | Unit of Analysis   |   Sample size |   Pageviews |
    |------------------+--------------------+---------------+-------------|
    | Gross conversion | click              |         25835 |      645875 |
    | Retention        | user-id(enrolled)  |         39115 |     4741213 |
    | Net conversion   | click              |         27413 |      685325 |


### Choosing Duration vs. Exposure

What percentage of Udacity's traffic would you divert to this experiment (assuming there were no other experiments you wanted to run simultaneously)? Is the change risky enough that you wouldn't want to run on all traffic?

Given the percentage you chose, how long would the experiment take to run, using the analytic estimates of variance? If the answer is longer than a few weeks, then this is unreasonably long, and you should reconsider an earlier decision.

---

Notice to evaluate multiple metrics in one experiment, we need to ensure that the experimental design meets the sample standards required for different metrics.

Recall the baseline unique cookies to view the course overview page per day is 40000. Practically, we collect the data using part of the traffic on the website in a period, since we should consider the experiment risk and accuracy. But, here I calculate the days need using 100% traffic on the website to give me a sense of what the duration looks like. Then I can think about how much traffic I might use in the experiment.


```python
print(tabulate([['Gross conversion','click',25835,math.ceil(25835/.08*2),math.ceil(25835/.08*2)/40000],
                ['Retention','user-id(enrolled)',39115,math.ceil(39115/(660/40000)*2),math.ceil(39115/(660/40000)*2)/40000],
                ['Net conversion','click',27413,math.ceil(27413/.08*2),math.ceil(27413/.08*2)/40000]],
               tablefmt='orgtbl',
               headers=['Metrics','Unit of Analysis','Sample size','Pageviews','Number of days (100% traffic)']))
```

    | Metrics          | Unit of Analysis   |   Sample size |   Pageviews |   Number of days (100% traffic) |
    |------------------+--------------------+---------------+-------------+---------------------------------|
    | Gross conversion | click              |         25835 |      645875 |                         16.1469 |
    | Retention        | user-id(enrolled)  |         39115 |     4741213 |                        118.53   |
    | Net conversion   | click              |         27413 |      685325 |                         17.1331 |


It is clear that the retention requires almost 119 days using 100% traffic to implement the experiment, which is a long period. On the other hand, gross conversion and net conversion only need almost 18 days.

Practically, we can decrease the duration by adjusting the related parameter:
    - increase the practical significance level
    - decrease statistical power 1−β
    - increase significance level α
Due to the limitations of the project requirement, I choose to abandon the retention metric. Then I choose to use pageviews required for net conversion as a benchmark. Of course, we can choose different percent of the traffic to implement the experiment due to your expectation of duration. Again, I will use 60% as the traffic exposed because of the limitations of the project.


```python
print(tabulate([['Gross conversion','click',25835,math.ceil(25835/.08*2),math.ceil(25835/.08*2)/(40000*.6)],
                ['Net conversion','click',27413,math.ceil(27413/.08*2),math.ceil(27413/.08*2)/(40000*.6)]],
               tablefmt='orgtbl',
               headers=['Metrics','Unit of Analysis','Sample size','Pageviews','Number of days (60% traffic)']))
```

    | Metrics          | Unit of Analysis   |   Sample size |   Pageviews |   Number of days (60% traffic) |
    |------------------+--------------------+---------------+-------------+--------------------------------|
    | Gross conversion | click              |         25835 |      645875 |                        26.9115 |
    | Net conversion   | click              |         27413 |      685325 |                        28.5552 |


To answer the question, I choose 685325 as the pageviews, 60% as traffic exposed and 29 days as duration to perform A/B test.

## 6.Analysis

The data for you to analyze is [here](https://docs.google.com/spreadsheets/d/1Mu5u9GrybDdska-ljPXyBjTpdZIUev_6i7t4LRDfXM8/edit#gid=0). This data contains the raw information needed to compute the above metrics, broken down day by day. Note that there are two sheets within the spreadsheet - one for the experiment group, and one for the control group.

The meaning of each column is:

**Pageviews:** Number of unique cookies to view the course overview page that day.
<br>
**Clicks:** Number of unique cookies to click the course overview page that day.
<br>
**Enrollments:** Number of user-ids to enroll in the free trial that day.
<br>
**Payments:** Number of user-ids who who enrolled on that day to remain enrolled for 14 days and thus make a payment. (Note that the date for this column is the start date, that is, the date of enrollment, rather than the date of the payment. The payment happened 14 days later. Because of this, the enrollments and payments are tracked for 14 fewer days than the other columns.)

---


```python
control=pd.read_csv('/Users/erjunqiu/Desktop/Udacity AB test/Final Project Results - Control.csv')
experiment=pd.read_csv('/Users/erjunqiu/Desktop/Udacity AB test/Final Project Results - Experiment.csv')
control_experiment=control.merge(experiment,how='inner',on='Date',suffixes=('_con','_exp'))
```

### Sanity Checks

Start by checking whether your invariant metrics are equivalent between the two groups. If the invariant metric is a simple count that should be randomly split between the 2 groups, you can use a binomial test. Otherwise, you will need to construct a confidence interval for a difference in proportions using a similar strategy as in Lesson 1, then check whether the difference between group values falls within that confidence level.


If your sanity checks fail, look at the day by day data and see if you can offer any insight into what is causing the problem.

---

Remember that the invariant metric will not change any matter in the control and experiment group. That is, assume our experiment set up properly, there is no difference in the metric between control and experiment from a statistical point of view.

**Number of cookies**


```python
# assume experiment set up properly
p=.5
# calculate total pageviews in experiment
n=control_experiment.loc[:,['Pageviews_con','Pageviews_exp']].sum().sum()
# caluculate total pageviews in control group (could be experiment group)
c=control_experiment.loc[:,'Pageviews_con'].sum()
# caluculate p_hat
p_hat=c/n
# calculate the standard error
se=math.sqrt(p*(1-p)/n)
# margin of error
me=se*1.96
# lower bound
lb=p-me
# upper bound
ub=p+me
# confidence interval
ci=[lb,ub]

if lb<=p_hat<=ub:
    sig='non-significant'
else:
    sig='significant'

print(tabulate([['p:',p],
                ['n:',n],
                ['p_hat:',p_hat],
                ['α:',.05],
                ['confidence interval:',ci],
                ['conclusion:',sig]],tablefmt='orgtbl',headers=['Number of cookies','parameters']))
```

    | Number of cookies    | parameters                                |
    |----------------------+-------------------------------------------|
    | p:                   | 0.5                                       |
    | n:                   | 690203                                    |
    | p_hat:               | 0.5006396668806133                        |
    | α:                   | 0.05                                      |
    | confidence interval: | [0.49882039214902313, 0.5011796078509769] |
    | conclusion:          | non-significant                           |


**Number of clicks**


```python
p=.5
n=control_experiment.loc[:,['Clicks_con','Clicks_exp']].sum().sum()
c=control_experiment.loc[:,'Clicks_con'].sum()
p_hat=c/n
se=math.sqrt(p*(1-p)/n)
me=se*1.96
lb=p-me
ub=p+me
ci=[lb,ub]

if lb<=p_hat<=ub:
    sig='non-significant'
else:
    sig='significant'

print(tabulate([['p:',p],
                ['p_hat:',p_hat],
                ['α:',.05],
                ['confidence interval:',ci],
                ['conclusion:',sig]],tablefmt='orgtbl',headers=['Number of clicks','parameters']))
```

    | Number of clicks     | parameters                                |
    |----------------------+-------------------------------------------|
    | p:                   | 0.5                                       |
    | p_hat:               | 0.5004673474066628                        |
    | α:                   | 0.05                                      |
    | confidence interval: | [0.49588449572378945, 0.5041155042762105] |
    | conclusion:          | non-significant                           |


**Click-through-probability**


```python
# ctp of control and experiment
ctp_con=control_experiment['Clicks_con'].sum()/control_experiment['Pageviews_con'].sum()
ctp_exp=control_experiment['Clicks_exp'].sum()/control_experiment['Pageviews_exp'].sum()
# expected ctp
ctp_diff=0
# calculate difference of ctp
ctp_diff_hat=ctp_con-ctp_exp
# calculate pooled ctp
ctp_pool=(control_experiment['Clicks_con'].sum()+
          control_experiment['Clicks_exp'].sum())/(control_experiment['Pageviews_con'].sum()+
                                                   control_experiment['Pageviews_exp'].sum())

se=math.sqrt(ctp_pool*(1-ctp_pool)*
             (1/control_experiment['Pageviews_con'].sum()+1/control_experiment['Pageviews_exp'].sum()))
me=se*1.96
lb=ctp_diff-me
ub=ctp_diff+me
ci=[lb,ub]

if lb<=ctp_diff_hat<=ub:
    sig='non-significant'
else:
    sig='significant'

print(tabulate([['d:',0],
                ['d̂:','%f'%ctp_diff_hat],
                ['α:',.05],
                ['confidence interval:',ci],
                ['conclusion:',sig]],tablefmt='orgtbl',headers=['Click-through-probability','parameters']))
```

    | Click-through-probability   | parameters                                      |
    |-----------------------------+-------------------------------------------------|
    | d:                          | 0                                               |
    | d̂:                          | -0.000057                                       |
    | α:                          | 0.05                                            |
    | confidence interval:        | [-0.0012956791986518956, 0.0012956791986518956] |
    | conclusion:                 | non-significant                                 |


All the invariant metrics have no significant differences between the control and experimental groups, so we pass the sanity check.

**It is worth mentioning that, if sanity check fails, you might need to drill down the metrics by days or other segments to debug the experiment setup.**

### Check for Practical and Statistical Significance

Next, for your evaluation metrics, calculate a confidence interval for the difference between the experiment and control groups, and check whether each metric is statistically and/or practically significance. A metric is statistically significant if the confidence interval does not include 0 (that is, you can be confident there was a change), and it is practically significant if the confidence interval does not include the practical significance boundary (that is, you can be confident there is a change that matters to the business.)


If you have chosen multiple evaluation metrics, you will need to decide whether to use the Bonferroni correction. When deciding, keep in mind the results you are looking for in order to launch the experiment. Will the fact that you have multiple metrics make those results more likely to occur by chance than the alpha level of 0.05?

---

**Gross conversion**


```python
valid_row=control_experiment['Enrollments_con'].notnull()
gc_con=control_experiment['Enrollments_con'].sum()/control_experiment.loc[valid_row,'Clicks_con'].sum()
gc_exp=control_experiment['Enrollments_exp'].sum()/control_experiment.loc[valid_row,'Clicks_exp'].sum()
gc_diff=0
gc_diff_hat=gc_exp-gc_con
gc_pool=(control_experiment['Enrollments_con'].sum()+
         control_experiment['Enrollments_exp'].sum())/(control_experiment.loc[valid_row,'Clicks_con'].sum()+
                                                       control_experiment.loc[valid_row,'Clicks_exp'].sum())

se=math.sqrt(gc_pool*(1-gc_pool)*
             (1/control_experiment.loc[valid_row,'Clicks_con'].sum()+
              1/control_experiment.loc[valid_row,'Clicks_exp'].sum()))
me=se*1.96
lb=(gc_diff_hat-gc_diff)-me
ub=(gc_diff_hat-gc_diff)+me
ci=[lb,ub]

if lb<=gc_diff<=ub:
    sig='non-significant'
else:
    sig='significant'
    
if ub<-.01 or lb>.01:
    pra='yes'
else:
    pra='no'
     
print(tabulate([['dmin:',.01],
                ['d:',gc_diff],
                ['d̂:',gc_diff_hat],
                ['α:',.05],
                ['confidence interval:',ci],
                ['conclusion:',sig],
                ['pratically signifiance',pra]],tablefmt='orgtbl'))
```

    | dmin:                  | 0.01                                        |
    | d:                     | 0                                           |
    | d̂:                     | -0.020554874580361565                       |
    | α:                     | 0.05                                        |
    | confidence interval:   | [-0.0291233583354044, -0.01198639082531873] |
    | conclusion:            | significant                                 |
    | pratically signifiance | yes                                         |


**Net conversion**


```python
valid_row=control_experiment['Payments_con'].notnull()
nc_con=control_experiment['Payments_con'].sum()/control_experiment.loc[valid_row,'Clicks_con'].sum()
nc_exp=control_experiment['Payments_exp'].sum()/control_experiment.loc[valid_row,'Clicks_exp'].sum()
nc_diff=0
nc_diff_hat=nc_exp-nc_con
nc_pool=(control_experiment['Payments_con'].sum()+
         control_experiment['Payments_exp'].sum())/(control_experiment.loc[valid_row,'Clicks_con'].sum()+
                                                    control_experiment.loc[valid_row,'Clicks_exp'].sum())

se=math.sqrt(nc_pool*(1-nc_pool)*
             (1/control_experiment.loc[valid_row,'Clicks_con'].sum()+
              1/control_experiment.loc[valid_row,'Clicks_exp'].sum()))
me=se*1.96
lb=(nc_diff_hat-nc_diff)-me
ub=(nc_diff_hat-nc_diff)+me
ci=[lb,ub]

if lb<=nc_diff<=ub:
    sig='non-significant'
    pra='no'    
else:
    sig='significant'

if lb>.0075 or ub<-.0075:
    pra='yes'
else:
    pra='no'

print(tabulate([['dmin:',.0075],
                ['d:',nc_diff],
                ['d̂:',nc_diff_hat],
                ['α:',.05],
                ['confidence interval:',ci],
                ['conclusion:',sig],
                ['pratically signifiance',pra]],tablefmt='orgtbl'))
```

    | dmin:                  | 0.0075                                        |
    | d:                     | 0                                             |
    | d̂:                     | -0.0048737226745441675                        |
    | α:                     | 0.05                                          |
    | confidence interval:   | [-0.011604624359891718, 0.001857179010803383] |
    | conclusion:            | non-significant                               |
    | pratically signifiance | no                                            |


Summarizing the results of the A/B test in one sentence is that gross conversion has significant differences in both practical and statistical, but net conversion is the opposite.

### Run Sign Tests

For each evaluation metric, do a sign test using the day-by-day breakdown. If the sign test does not agree with the confidence interval for the difference, see if you can figure out why.

---

You can do a sign test calculation using this [online calculator](https://www.graphpad.com/quickcalcs/binomial1.cfm).

We use the binomial distribution to calculate how likely the experiment side does have a difference with the control side.


```python
control_experiment['Gross conversion_con']=control_experiment['Enrollments_con']/control_experiment['Pageviews_con']
control_experiment['Gross conversion_exp']=control_experiment['Enrollments_exp']/control_experiment['Pageviews_exp']
control_experiment['Net conversion_con']=control_experiment['Payments_con']/control_experiment['Pageviews_con']
control_experiment['Net conversion_exp']=control_experiment['Payments_exp']/control_experiment['Pageviews_exp']

gc_n=control_experiment['Gross conversion_con'].count()
gc_success=(control_experiment['Gross conversion_con']<control_experiment['Gross conversion_exp']).sum()
nc_n=control_experiment['Net conversion_con'].count()
nc_success=(control_experiment['Net conversion_con']<control_experiment['Net conversion_exp']).sum()

print(tabulate([['Gross conversion',gc_success,gc_n],
                ['Net conversion',nc_success,nc_n]],tablefmt='orgtbl',headers=['metric','success','n']))
```

    | metric           |   success |   n |
    |------------------+-----------+-----|
    | Gross conversion |         4 |  23 |
    | Net conversion   |        10 |  23 |



```python
print(tabulate([['Gross conversion',.0026,.05,'significant'],
                ['Net conversion',.6776,.05,'non-significant']],
               tablefmt='orgtbl',
               headers=['metric','p-value','subject','conclusion']))
```

    | metric           |   p-value |   subject | conclusion      |
    |------------------+-----------+-----------+-----------------|
    | Gross conversion |    0.0026 |      0.05 | significant     |
    | Net conversion   |    0.6776 |      0.05 | non-significant |


#### Bonferroni correction consideration

Will you use the Bonferroni correction in your analysis phase? State whether you used the Bonferroni correction, and explain why or why not. If there are any discrepancies between the effect size hypothesis tests and the sign tests, describe the discrepancy and why you think it arose.

---

The short answer is no.
The long answer:
Bonferroni correction mainly useful when there are a fairly small number of multiple comparisons and you're looking for one or two that might be significant (John H. McDonald). The use of Bonferroni correction determines whether you only expect certain metrics to be statistically significant in multiple comparisons. Simply put, it is the choice between 'all' and 'any' since Bonferroni correction increases the probability of the second type of error (false negative) while reducing the probability of the first type of error (false positive).

Our expectation of the metrics change is:
    - Gross conversion decreases significantly
    - Net conversion does not decrease significantly

In this case, we expect the two metrics to be statistically significant in the direction we are looking for, so I choose 'all'.

Both metrics have the same conclusion between effect size hypothesis test and sign test.

### Make a Recommendation

Finally, make a recommendation. Would you launch this experiment, not launch it, dig deeper, run a follow-up experiment, or is it a judgment call? If you would dig deeper, explain what area you would investigate. If you would run follow-up experiments, briefIy describe that experiment. If it is a judgment call, explain what factors would be relevant to the decision.

---

The gross conversion has statistically and practically significant differences between control and experiment, and the experimental group reflects the decline in the number of enrolled users as we expected. 

There is no statistically significant difference in net conversion between control and experiment, which seems to be consistent with our expectations. Note that the confidence interval includes positive and negative values, and the lower bound is lower than the practical confidence level.

My recommendation is that do not launch the new feature followed by two considerations:
    - the experiment net conversion may decline practically significant which opposite our expectation
    - the profit may be negatively affected, as the difference's range of the net conversion includes negative values

It is important for Udacity to evaluate if there is really a practically financial loss to evaluate if the feature is worth to launch. In addition, Udacity might do research focus on the pop-up screener text description to figure out if the content decreases the number of students who continue past the free trial and eventually complete the course.

## Follow-Up Experiment: How to Reduce Early Cancellations

If you wanted to reduce the number of frustrated students who cancel early in the course, what experiment would you try? Give a brief description of the change you would make, what your hypothesis would be about the effect of the change, what metrics you would want to measure, and what unit of diversion you would use. Include an explanation of each of your choices.

---

Our goal is to reduce the number of frustrated students, I would recommend product team add a feature if the student clicked "cancel enrollment", they will see a pop-up screen lists some appropriate articles or usual questions keywords that recapture student interest. For example, relevant basic knowledge, time management, and course experience sharing can all appear as content in the pop-up screen. At this point, the student would have the option to click into a link or continue canceling the enrollment.

The hypothesis was that this might help students re-establish interest and confidence, thus reducing the number of frustrated students who left the enrollment because they are frustrated. If this hypothesis held true, Udacity could improve the overall students' experience and support students to complete the course successfully, increase the number of students who make the payment.

The unit of diversion is a user-ids. For users that do not enroll, their user-id is not tracked in the experiment, even if they were signed in when they visited the course overview page.

**Invariant Metrics:**
<br>
Number of user-ids: That is, number of users who enroll in the course.
<br>
Number of clicks: That is, number of unique user-ids to click the "Cancel enrollment" button (which happens before the recapture screener is triggered).
<br>
Click-through-probability: That is, number of unique user-ids to click the "Cancel enrollment" button divided by number of unique user-ids to enroll in the course.

**Evaluation Metrics:**
<br>
Cancel rate: That is, number of user-ids to cancel enrollment divided by number of user-ids to enroll in course.

### Reference

*Udacity A/B Testing* - https://www.udacity.com/course/ab-testing--ud257
<br>
*How to choose the right UX metrics for your product* - https://library.gv.com/how-to-choose-the-right-ux-metrics-for-your-product-5f46359ab5be
<br>
*Multiple comparisons* - http://www.biostathandbook.com/multiplecomparisons.html
