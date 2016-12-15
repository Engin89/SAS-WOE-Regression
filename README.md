# SAS-WOE-Regression
FILENAME TRAIN '/folders/myfolders/sasuser.v94/train.csv';
PROC IMPORT
DATAFILE= TRAIN
DBMS=CSV
OUT= WORK.MYCSV ;
GETNAMES=YES;
RUN;
FILENAME TEST '/folders/myfolders/sasuser.v94/test.csv';
PROC IMPORT
DATAFILE= TEST
DBMS=CSV
OUT= WORK.MYCSV_TEST ;
GETNAMES=YES;
RUN;
proc means data = work.mycsv;
class Sex;
var Pclass Age Fare Parch;
run;
proc freq data=work.mycsv(drop= Age Fare Parch);
tables Embarked;
tables Pclass;
run;
proc univariate data = work.mycsv;
class Sex;
var Age Fare Parch;
histogram / kernel /*Pclass Age Fare Parch*/;
run;
proc sql;
create table work.titanic_sex as select
	Sex,
	sum(Survived)/count(Survived) as Survive_Rate from WORK.MYCSV group by Sex;
run;
create table work.titanic_embarked as select
	Embarked,
	sum(Survived)/count(Survived) as Survive_Rate from WORK.MYCSV group by Embarked;
run;
create table work.titanic_pclass as select
	Pclass,
	sum(Survived)/count(Survived) as Survive_Rate from WORK.MYCSV group by Pclass;
run;
%macro scatter();
%let varlist = Age Fare Parch;
%let i = 1;
%do %while (&i <= 3);
	%let j = %sysevalf(&i+1);
	%do %while(&j <= 3);
	 %if &i ne &j %then %do;
	 	%let x_used=%qscan(%bquote(&varlist),&i);
	 	%let y_used=%qscan(%bquote(&varlist),&j);
		proc template;
  			define statgraph scatterplot;
    			begingraph; 
      				entrytitle "&x_used and &y_used by Survived"; 
      					layout overlay; 	 
       						 scatterplot x=&x_used y=&y_used / 
          					 group=Survived name="scatter" datalabel=Survived;
        					 discretelegend "scatter";
      					endlayout;	
    			endgraph;
    			end;
    			proc sgrender data=work.mycsv template=scatterplot;
run;
      %end;
    %let j =%sysevalf(&j + 1);
	%end;
%let i = %sysevalf(&i + 1);
%end;
%mend scatter;
%scatter();
%macro boxplot(file=,varlist=);
%let nitems = %sysfunc(countw(&varlist));
%do i=1 %to &nitems;
			%let x_used=%qscan(%bquote(&varlist),&i);
			proc sort data= work.&file. out=work.&file._sorted;
			by &x_used;
			run;
	 		proc boxplot data = work.&file._sorted;
  			title1 'Boxplot of Survived vs '&x_used.'';
  			plot Survived*&x_used /boxconnect=mean boxstyle=schematic haxis=axis1 vaxis = axis2;
			run;
%end;
%mend boxplot;
proc print data=mycsv_test;
run;
/*bucketing*/
data bucketed_test(drop= Name Cabin Ticket);
set work.mycsv_test;
format Parch_Bucket $7.;
if Parch lt 1 then Parch_Bucket = '0';
else if Parch lt 2 then Parch_Bucket =  '1-2';
else Parch_Bucket =  '3-';
if Embarked eq "" then Embarked = 'C';

SibSp = Min(SibSp,4);
format Age_Bucket $7.;
if age eq . then Age_Bucket = 'Missing';
else if age lt 8 then Age_Bucket = '[0-8)';
else if age lt 16 then Age_Bucket =  '[08-16)';
else if age lt 24 then Age_Bucket =  '[16-24)';
else if age lt 32 then Age_Bucket =  '[24-32)';
else if age lt 40 then Age_Bucket =  '[32-40)';
else if age lt 48 then Age_Bucket =  '[40-48)';
else if age lt 56 then Age_Bucket =  '[48-56)';
else Age_Bucket = '[56-)';
format Fare_Bucket $8.;
if Fare eq . then Fare_Bucket = 'Missing';
else if Fare lt 7.5 then Fare_Bucket = '[0-7.5)';
else if Fare lt 10 then Fare_Bucket = '[07.5-10)';
else if Fare lt 12 then Fare_Bucket = '[10-12)';
else if Fare lt 15 then Fare_Bucket = '[12-15)';
else if Fare lt 20 then Fare_Bucket = '[15-20)';
else if Fare lt 25 then Fare_Bucket = '[20-25)';
else if Fare lt 30 then Fare_Bucket = '[20-30)';
else if Fare lt 40 then Fare_Bucket = '[30-40)';
else if Fare lt 60 then Fare_Bucket = '[40-60)';
else if Fare lt 100 then Fare_Bucket = '[60-100)';
else Fare_Bucket = '[612)';
run;


data bucketed_train(drop= Name Cabin Ticket);
set work.mycsv;
format Parch_Bucket $7.;
if Parch lt 1 then Parch_Bucket = '0';
else if Parch lt 2 then Parch_Bucket =  '1-2';
else Parch_Bucket =  '3-';
if Embarked eq "" then Embarked = 'C';

SibSp = Min(SibSp,4);
format Age_Bucket $7.;
if age eq . then Age_Bucket = 'Missing';
else if age lt 8 then Age_Bucket = '[0-8)';
else if age lt 16 then Age_Bucket =  '[08-16)';
else if age lt 24 then Age_Bucket =  '[16-24)';
else if age lt 32 then Age_Bucket =  '[24-32)';
else if age lt 40 then Age_Bucket =  '[32-40)';
else if age lt 48 then Age_Bucket =  '[40-48)';
else if age lt 56 then Age_Bucket =  '[48-56)';
else Age_Bucket = '[56-)';
format Fare_Bucket $8.;
if Fare eq . then Fare_Bucket = 'Missing';
else if Fare lt 7.5 then Fare_Bucket = '[0-7.5)';
else if Fare lt 10 then Fare_Bucket = '[07.5-10)';
else if Fare lt 12 then Fare_Bucket = '[10-12)';
else if Fare lt 15 then Fare_Bucket = '[12-15)';
else if Fare lt 20 then Fare_Bucket = '[15-20)';
else if Fare lt 25 then Fare_Bucket = '[20-25)';
else if Fare lt 30 then Fare_Bucket = '[20-30)';
else if Fare lt 40 then Fare_Bucket = '[30-40)';
else if Fare lt 60 then Fare_Bucket = '[40-60)';
else if Fare lt 100 then Fare_Bucket = '[60-100)';
else Fare_Bucket = '[612)';
run;

proc freq data=bucketed_train;
tables Age_Bucket;
tables Fare_Bucket;
run;
%boxplot(file = mycsv,varlist = Embarked Pclass Sex);
%boxplot(file=bucketed_train, varlist = Age_Bucket Fare_Bucket);
%macro woe_iv_calc(file=,varlist=);
%let nitems = %sysfunc(countw(&varlist));
%do i=1 %to &nitems;
	%let response = %qscan(%bquote(&varlist),&i);
	proc sql;
	create table work.&response. as select
	PassengerId,
	&Response,
	Survived
	from work.&file.;
	quit;
	proc sql;
	select sum(Survived) into: event_total from work.&response.;quit;
	proc sql;
	select count(Survived) into: event_count from work.&response.;quit;
	%let nonevent_total = %sysevalf(&event_count - &event_total);
	proc sql;
	create table work.&response. as select
	&Response,
	sum(Survived)/&event_total as Event_Rate,
	(count(Survived) - sum(Survived))/&nonevent_total as Nonevent_Rate,
	log(((count(Survived) - sum(Survived))/&nonevent_total)/(sum(Survived)/&event_total)) as WOE
	from work.&response.
	group by &Response;
	quit;
	proc sql;
	create table work.&response. as select
	&Response,
	Event_Rate,
	Nonevent_Rate,
	WOE,
	(Nonevent_Rate-Event_Rate)*WOE as IV
	from work.&response.
	group by &Response;
	quit;
	/*proc sql;
	create table work.&response. as select
	sum(iv) as final_iv
	from work.&response.;
	quit;*/
%end;
%mend woe_iv_calc;
%woe_iv_calc(file = bucketed_train,varlist = Sex Embarked Pclass Parch_Bucket SibSp Age_Bucket Fare_Bucket );
proc sql;
create table work.woe_added_train as select
x.PassengerId,
x.Age_Bucket,
x.Fare_Bucket,
x.Survived,
x.SibSp,
x.Embarked,
x.Sex,
y.WOE as Age_Bucket_WOE,
z.WOE as Fare_Bucket_WOE,
n.WOE as SibSp_WOE,
k.WOE as Embarked_WOE,
l.WOE as Sex_WOE,
p.WOE as Pclass_WOE
from bucketed_train x left join work.Age_Bucket y on (x.Age_Bucket=y.Age_Bucket)
left join work.Fare_Bucket z on (x.Fare_Bucket = z.Fare_Bucket)
left join work.SibSP n on (x.SibSp = n.SibSp)
left join work.Embarked k on (x.Embarked = k.Embarked)
left join work.Sex l on (x.Sex = l.Sex)
left join work.Pclass p on (x.Pclass=p.Pclass); quit;
proc sql;
create table work.woe_added_test as select
x.PassengerId,
x.Age_Bucket,
x.Fare_Bucket,
x.SibSp,
x.Embarked,
x.Sex,
y.WOE as Age_Bucket_WOE,
z.WOE as Fare_Bucket_WOE,
n.WOE as SibSp_WOE,
k.WOE as Embarked_WOE,
l.WOE as Sex_WOE,
p.WOE as Pclass_WOE
from bucketed_test x left join work.Age_Bucket y on (x.Age_Bucket=y.Age_Bucket)
left join work.Fare_Bucket z on (x.Fare_Bucket = z.Fare_Bucket)
left join work.SibSP n on (x.SibSp = n.SibSp)
left join work.Embarked k on (x.Embarked = k.Embarked)
left join work.Sex l on (x.Sex = l.Sex)
left join work.Pclass p on (x.Pclass=p.Pclass); quit;

proc corr data=work.woe_added_train;
var Sex_woe Embarked_woe Pclass_woe SibSp_woe Age_Bucket_woe Fare_Bucket_woe;
run;
proc logistic data=work.woe_added_train nocov outest=survival_model;
model Survived(event = "1") = Embarked_woe Age_Bucket_woe SibSp_woe Fare_Bucket_woe Pclass_woe Sex_woe
/selection= stepwise slentry=0.1 slstay=0.25 details lackfit sequential  outroc=work.roc_input ctable  ;
output out=work.woe_added_train_scrd  p=phat lower=lcl upper=ucl
             predprobs=(individual crossvalidate);
score data=work.woe_added_test out=test_data_scored outroc= outroc_Test;
run;
proc reg data = work.woe_added_train;
model Survived = Embarked_woe Age_Bucket_woe SibSp_woe Fare_Bucket_woe Pclass_woe Sex_woe/ vif tol;
run;
proc sql;
create table final_predictions as select
PassengerId,
I_Survived 
from test_data_scored;
run;
proc sql;
create table boosting_woe_train as select
PassengerId,
Age_Bucket_WOE as X1,
Fare_Bucket_WOE as X2,
SibSp_WOE as X3,
Embarked_WOE as X4,
Sex_WOE as X5,
IP_1 as X6,
Survived as Y from work.woe_added_train_scrd; quit;







