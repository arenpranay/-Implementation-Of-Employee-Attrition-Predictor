# -Implementation-Of-Employee-Attrition-Predictor
import pandas as pd

import numpy as np

# Commented out IPython magic to ensure Python compatibility.
import matplotlib.pyplot as plt
# %matplotlib inline

import seaborn as sns

data = pd.read_csv('Employee_Attrition.csv')

data.head()

data.describe()

data.corr()

data.drop('EmployeeCount',axis=1, inplace = True)
data.drop('StandardHours',axis=1, inplace = True)

data.head()

data.drop('EmployeeNumber', inplace = True, axis =1)

data.isnull().sum()

data.dtypes

sns.distplot(data['Age'])

fig,ax = plt.subplots(3,3, figsize=(12,12))               # 'ax' has references to all the four axes
sns.distplot(data['TotalWorkingYears'], ax = ax[0,0]) 
sns.distplot(data['YearsAtCompany'], ax = ax[0,1]) 
sns.distplot(data['DistanceFromHome'], ax = ax[0,2]) 
sns.distplot(data['YearsInCurrentRole'], ax = ax[1,0]) 
sns.distplot(data['YearsWithCurrManager'], ax = ax[1,1]) 
sns.distplot(data['YearsSinceLastPromotion'], ax = ax[1,2]) 
sns.distplot(data['PercentSalaryHike'], ax = ax[2,0]) 
sns.distplot(data['YearsSinceLastPromotion'], ax = ax[2,1]) 
sns.distplot(data['TrainingTimesLastYear'], ax = ax[2,2]) 
plt.show()

data['DailyRate'].hist(bins=20)

sns.countplot(x= data['Attrition'], data = data, hue = data['Gender'])

sns.countplot(x = data['MaritalStatus'],hue=data['Attrition'] , data = data)

sns.barplot(y=data['JobRole'], x=data['JobSatisfaction'],estimator = np.mean, data = data)

"""Label Encoding"""

from sklearn.preprocessing import LabelEncoder

cat_object= ['Attrition', 'BusinessTravel', 'Department','EducationField','Gender','JobRole','MaritalStatus','Over18','OverTime']

le = LabelEncoder()

for obj in cat_object:
    data[obj] = le.fit_transform(data[obj])
data.dtypes

corr = data.corr()
f,ax = plt.subplots(figsize=(16,9))
sns.heatmap(corr, xticklabels = corr.columns.values, yticklabels = corr.columns.values, square=True)

data.drop(['Over18'], axis=1, inplace= True)
k=10
cols=corr.nlargest(k,'Attrition')['Attrition'].index
cm= np.corrcoef(data[cols].values.T)
sns.set(font_scale=1.25)
hm = sns.heatmap(cm, cbar = True ,annot = True,fmt ='.2f',annot_kws ={'size':10}, yticklabels=cols.values, xticklabels=cols.values )
plt.show()

#Corelation of variables with Attrition
total_records= len(data)
columns = ["OverTime","MaritalStatus","JobRole","Department",'NumCompaniesWorked',"Gender","EducationField"]

j=0
for i in columns:
    j +=1
    plt.subplot(4,2,j)
    ax1 = sns.countplot(data=data,x= i,hue="Attrition",palette="Set2")
    if(j==9 or j== 10):
        plt.xticks( rotation=90)
    for p in ax1.patches:
        height = p.get_height()
        #ax1.text(p.get_x()+p.get_width()/2.,
               # height + 3,
                #'{:1.2f}'.format(height/total_records,0),
                #ha="center",rotation=0) 

# Custom the subplot layout
plt.subplots_adjust(bottom=0.3, top=4)
plt.show()

sns.countplot(data.Attrition)
import statsmodels.api as sm

x = data.drop(['Attrition'],axis=1)
y = data['Attrition']

X_1 = sm.add_constant(x)

"""Calculating pvalues"""

model = est = sm.OLS(y,X_1).fit()
k = model.pvalues
k

"""Feature Selection on the basis of pvalues"""

cols = list(x.columns)
pmax = 1
while (len(cols)>0):
    p =[]
    x_1 = x[cols]
    x_1 = sm.add_constant(x_1)
    model = sm.OLS(y,x_1).fit()
    p= pd.Series(model.pvalues.values[1:],index = cols)
    pmax = max(p)
    feature_with_p_max = p.idxmax()
    if (pmax>0.05):
        cols.remove(feature_with_p_max)
    else:
        break

selected_ft = cols
selected_ft

#FEATURES SELECTED ON THE BASIS OF P VALUES
num = p.values
val = p.index
sns.barplot(x=num,y = val)

data_new = data[selected_ft]
data_new.head()

X=data_new
X.shape
X.head()

Y=y
Y.shape
Y.head()

"""SMOTE Analysis"""

print("Before OverSampling, counts of label '1': {}".format(sum(Y== 1))) 
print("Before OverSampling, counts of label '0': {} \n".format(sum(Y == 0))) 
  
# import SMOTE module from imblearn library 
# pip install imblearn (if you don't have imblearn in your system) 
from imblearn.over_sampling import SMOTE 
sm = SMOTE(random_state = 2) 
X_train_res, y_train_res = sm.fit_sample(X, Y.ravel()) 
  
print('After OverSampling, the shape of train_X: {}'.format(X.shape)) 
print('After OverSampling, the shape of train_y: {} \n'.format(Y.shape)) 
  
print("After OverSampling, counts of label '1': {}".format(sum(y_train_res == 1))) 
print("After OverSampling, counts of label '0': {}".format(sum(y_train_res == 0)))

#BALANCED CLASS THROUGH SMOT ANALYSIS
sns.countplot(y_train_res)

"""Train Test Split"""

from sklearn.model_selection import train_test_split
x_train, x_test, y_train, y_test = train_test_split(X_train_res,y_train_res, test_size=0.30, random_state =36)

"""Random Forest Classifier"""

from sklearn.ensemble import RandomForestClassifier
classifier = RandomForestClassifier(n_estimators = 46, criterion ='entropy',max_features='auto',random_state =36)
classifier.fit(x_train,y_train)

from sklearn.model_selection import GridSearchCV
from sklearn.ensemble import RandomForestClassifier
n_estimators = list(range(1,50))
#Convert to dictionary
hyperparameters = dict(n_estimators=n_estimators)
classifier1= RandomForestClassifier()
#Use GridSearch
clf = GridSearchCV(classifier1, hyperparameters, cv=10)

#Fit the model
clf.fit(x_train,y_train)

pred=clf.predict(x_test)
from sklearn.metrics import confusion_matrix,classification_report
print(classification_report(y_test, pred))

#FEATURE IMPORTANCE ACCORDING TO RANDOM FOREST CLASSIFIER
from sklearn.datasets import make_regression
# define dataset
features=selected_ft
importance = classifier.feature_importances_
# summarize feature importance
for i,v in enumerate(importance):
	print('Feature: %0d, Score: %.5f' % (i,v))
# plot feature importance
indices = np.argsort(importance)
plt.figure(figsize=(12,12))
plt.title("Feature Importances")
plt.barh(range(len(indices)), importance[indices], color="green")
plt.yticks(range(len(indices)), [features[i] for i in indices])
plt.xlabel("Relative Importance")
plt.show()

from sklearn.metrics import confusion_matrix

data = confusion_matrix(y_test,pred)
df_cm = pd.DataFrame(data, columns=np.unique(y_test), index = np.unique(y_test))
df_cm.index.name = 'Actual'
df_cm.columns.name = 'Predicted'
plt.figure(figsize = (6,6))
sns.set(font_scale=1.4)#for label size
sns.heatmap(df_cm, annot=True,annot_kws={"size": 12})# font size

"""Logistic Regression"""

from sklearn.linear_model import LogisticRegression 
model = LogisticRegression(random_state=0)
model.fit(x_train,y_train)
pred1=model.predict(x_test)
pred1

print(classification_report(y_test, pred1))

from sklearn.metrics import confusion_matrix

data = confusion_matrix(y_test, pred1)
df_cm = pd.DataFrame(data, columns=np.unique(y_test), index = np.unique(y_test))
df_cm.index.name = 'Actual'
df_cm.columns.name = 'Predicted'
plt.figure(figsize = (6,6))
sns.set(font_scale=1.4)#for label size
sns.heatmap(df_cm, annot=True,annot_kws={"size": 12})# font size

"""Decision Tree Classifier"""

from sklearn.tree import DecisionTreeClassifier
classifier_dec = DecisionTreeClassifier(criterion = 'entropy',random_state=0)
classifier_dec.fit(x_train,y_train)
y_dec= classifier_dec.predict(x_test)
print(classification_report(y_test, y_dec))

features = selected_ft
importances = classifier_dec.feature_importances_
indices = np.argsort(importances)
plt.figure(figsize=(12,12))
plt.title("Feature Importances")
plt.barh(range(len(indices)), importances[indices], color="green")
plt.yticks(range(len(indices)), [features[i] for i in indices])
plt.xlabel("Relative Importance")
plt.show()

from sklearn.metrics import confusion_matrix

data = confusion_matrix(y_test, y_dec)
df_cm = pd.DataFrame(data, columns=np.unique(y_test), index = np.unique(y_test))
df_cm.index.name = 'Actual'
df_cm.columns.name = 'Predicted'
plt.figure(figsize = (6,6))
sns.set(font_scale=1.4)#for label size
sns.heatmap(df_cm, cmap="Blues", annot=True,annot_kws={"size": 12})# font size

"""KNN Classifier"""

from sklearn.neighbors import KNeighborsClassifier
neigh = KNeighborsClassifier(n_neighbors=2)

neigh.fit(x_train,y_train)

kpred=neigh.predict(x_test)
kpred

print(classification_report(y_test,kpred))

from sklearn.impute import SimpleImputer
imputer = SimpleImputer(missing_values= np.nan, strategy= 'mean')
imputer = imputer.fit(X_train_res)
XX = imputer.transform(X_train_res)

from sklearn.neighbors import KNeighborsClassifier
from sklearn.model_selection import cross_val_score
n_feats = X.shape[1]
print('Feature  Accuracy')
for i in range(n_feats):
    X1 = XX[:, i].reshape(-1, 1)
    scores = cross_val_score(neigh, X1, y_train_res, cv=3)
    print(f'{i}        {scores.mean():g}')

from sklearn.metrics import confusion_matrix

data = confusion_matrix(y_test, kpred)
df_cm = pd.DataFrame(data, columns=np.unique(y_test), index = np.unique(y_test))
df_cm.index.name = 'Actual'
df_cm.columns.name = 'Predicted'
plt.figure(figsize = (6,6))
sns.set(font_scale=1.4)#for label size
sns.heatmap(df_cm, cmap="Blues", annot=True,annot_kws={"size": 12})# font size

#error for knn
error_rate = []


for i in range(1,30):
    
    knn = KNeighborsClassifier(n_neighbors=i)
    knn.fit(x_train,y_train)
    pred_i = knn.predict(x_test)
    error_rate.append(np.mean(pred_i != y_test))

plt.figure(figsize=(10,6))
plt.plot(range(1,30),error_rate,color='blue', linestyle='dashed', marker='o',
         markerfacecolor='red', markersize=10)
plt.title('Error Rate vs. K Value')
plt.xlabel('K')
plt.ylabel('Error Rate')

"""Comparing the models"""

import warnings
warnings.filterwarnings("ignore")
    # load libraries
import matplotlib.pyplot as plt
from sklearn import model_selection
from sklearn.linear_model import LogisticRegression
from sklearn.tree import DecisionTreeClassifier
from sklearn.neighbors import KNeighborsClassifier
     
    
from sklearn.svm import SVC
from sklearn.model_selection import train_test_split
from sklearn import datasets
import matplotlib.pyplot as plt
plt.style.use('ggplot')
     
seed = 42
    #dataset = datasets.load_breast_cancer()
    #X = dataset.data; y = dataset.target
    #X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.25)
kfold = model_selection.KFold(n_splits=10, random_state=seed)
    # prepare models
models = []
models.append(('LR', LogisticRegression()))
models.append(('RFC', RandomForestClassifier(bootstrap=True, ccp_alpha=0.0, class_weight=None,
                       criterion='entropy', max_depth=None, max_features='auto',
                       max_leaf_nodes=None, max_samples=None,
                       min_impurity_decrease=0.0, min_impurity_split=None,
                       min_samples_leaf=1, min_samples_split=2,
                       min_weight_fraction_leaf=0.0, n_estimators=46,
                       n_jobs=None, oob_score=False, random_state=36, verbose=0,
                       warm_start=False)))
models.append(('KNN', KNeighborsClassifier(algorithm='auto', leaf_size=30, metric='minkowski',
                     metric_params=None, n_jobs=None, n_neighbors=2, p=2,
                     weights='uniform')))
models.append(('DTC', DecisionTreeClassifier()))
    
    # evaluate each model
results = []
names = []
scoring = 'accuracy'
for name, model in models:
	  kfold = model_selection.KFold(n_splits=10, random_state=seed)
	  cv_results = model_selection.cross_val_score(model, x_train, y_train, cv=kfold, scoring=scoring)
	  results.append(cv_results)
	  names.append(name)
	  msg = "%s: %f (%f)" % (name, cv_results.mean(), cv_results.std())
	  print(msg)
    # boxplot algorithm comparison
fig = plt.figure(figsize=(10,10))
fig.suptitle('Compare sklearn classification algorithms')
ax = fig.add_subplot(111)
plt.boxplot(results)
ax.set_xticklabels(names)
plt.show()

import pickle
pickle.dump(neigh,open('model.pkl','wb'))

model=pickle.load(open('model.pkl','rb'))
