## Ryan Solution 

출처 :  https://dacon.io/competitions/official/21265/codeshare/438 

**코드를 깔끔하게 짜는 것이 배울 점**

- 파생 변수는 우리팀의 접근방법과 똑같고 모델부분도 똑같음
- 코드를 함수를 만들어서 깔끔하게 짠것을 배우면 좋을 듯 

```python
def get_specific_mean(var_names,train,test,chunk_var=[],bins_num=[]):
    train['index']=1
    test['index']=0
    data=pd.concat([train,test],axis=0)
    if(len(chunk_var)!=0):
        for i,j in zip(chunk_var,bins_num):
            data[i+"chunk"]=pd.cut(data[i],bins=j)
    
    var_names.extend([x+"chunk" for x in chunk_var])
    train=data.loc[data['index']==1,]
    test=data.loc[data['index']==0,]
    apartment_id_count=train.groupby(var_names)['transaction_real_price'].std().reset_index(name="_".join(var_names)+" "+"-".join([str(x) for x in bins_num])+" "+'std')
    apartment_id_count=pd.merge(apartment_id_count,train.groupby(var_names).size().reset_index(name="_".join(var_names)+" "+"-".join([str(x) for x in bins_num])+" "+'count'),on=var_names,how='left')
    apartment_id_count=pd.merge(apartment_id_count,train.groupby(var_names)['transaction_real_price'].mean().reset_index(name="_".join(var_names)+" "+"-".join([str(x) for x in bins_num])+" "+'mean'),on=var_names,how='left')
    apartment_id_count=pd.merge(apartment_id_count,train.groupby(var_names)['transaction_daydiff'].std().reset_index(name="_".join(var_names)+" "+"-".join([str(x) for x in bins_num])+" "+'daydiff std'),on=var_names,how='left')
    
    train=pd.merge(train,apartment_id_count,on=var_names,how='left')
    train=train.drop([x+"chunk" for x in chunk_var],axis=1)
    train=train.drop(['index'],axis=1)
    
    train["_".join(var_names)+" "+"-".join([str(x) for x in bins_num])+" "+'mean']=np.where(train["_".join(var_names)+" "+"-".join([str(x) for x in bins_num])+" "+'count']==1,np.nan,(train["_".join(var_names)+" "+"-".join([str(x) for x in bins_num])+" "+'mean']*train["_".join(var_names)+" "+"-".join([str(x) for x in bins_num])+" "+'count']-train['transaction_real_price'])/(train["_".join(var_names)+" "+"-".join([str(x) for x in bins_num])+" "+'count']-1))
    #train["_".join(var_names)+" "+"-".join([str(x) for x in bins_num])+" "+'daydiff mean']=np.where(train["_".join(var_names)+" "+"-".join([str(x) for x in bins_num])+" "+'count']==1,np.nan,(train["_".join(var_names)+" "+"-".join([str(x) for x in bins_num])+" "+'daydiff mean']*train["_".join(var_names)+" "+"-".join([str(x) for x in bins_num])+" "+'count']-train['transaction_daydiff'])/(train["_".join(var_names)+" "+"-".join([str(x) for x in bins_num])+" "+'count']-1))
    
    test=pd.merge(test,apartment_id_count,on=var_names,how='left')
    test=test.drop([x+"chunk" for x in chunk_var],axis=1)
    test=test.drop(['index'],axis=1)

    return train,test
```

```python
agg_funs={
    'foundation_date':['mean','min','max'],
    'school_class_elementary':['sum'],
    'school_class_high':['sum'],
    'school_class_middle':['sum'],
    'operation_type_national':['sum'],
    'operation_type_private':['sum'],
    'operation_type_public':['sum'],
    'highschool_type_autonomous':['sum'],
    'highschool_type_general':['sum'],
    'highschool_type_objective':['sum'],
    'highschool_type_specialized':['sum'],
    'gender_both':['sum'],
    'gender_female':['sum'],
    'gender_male':['sum']
}

school=school.groupby(['address_by_law']).agg(agg_funs).reset_index(drop=False)
```

