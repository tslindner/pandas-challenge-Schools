
<big><big><b>PyCity Schools Analysis</b></big></big>  <br>

OBSERVED TREND 1:  Charter schools perform better than district shcools  <br>
OBSERVED TREND 2:  The performance of a school's students is inversely related to the size of the school.  <br>
OBSERVED TREND 3:  The data suggests that the performance of a school's students is inversley related to the school's spending per student.  However, it's possible that this is simply because larger shcools are more expensive, and larger schools perform worse.  <br>


```python
import pandas as pd

schools_inpath = 'raw_data/schools_complete.csv'
students_inpath = 'raw_data/students_complete.csv'

schools_df = pd.read_csv(schools_inpath, dtype={'School ID' : int, 
                                                'name' : object, 
                                                'type' : object, 
                                                'size' : int, 
                                                'budget' : int})

students_df = pd.read_csv(students_inpath, dtype = {'Student ID' : int,
                                                    'name' : object,
                                                    'gender' : object,
                                                    'grade' : object,
                                                    'school' : object,
                                                    'reading_score' : int,
                                                    'math_score' : int})

```

<big><b>District Summary</b></big>


```python
total_schools = schools_df['name'].count()
total_students = students_df['name'].count()
total_budget = schools_df['budget'].sum()
average_math_score = students_df['math_score'].sum() / total_students
average_reading_score = students_df['reading_score'].sum() / total_students

passing_students_math = students_df.loc[students_df['math_score'] > 60, :]['name'].count()
passing_rate_math = (passing_students_math / total_students) * 100

passing_students_reading = students_df.loc[students_df['reading_score'] > 60, :]['name'].count()
passing_rate_reading = (passing_students_reading / total_students) * 100

overall_passing_rate = (passing_rate_math + passing_rate_reading) / 2

district_summary_df = pd.DataFrame(
        {
            'Total Schools' : total_schools,
            'Total Students' : total_students,
            'Total Budget' : total_budget,
            'Average Math Score' : average_math_score,
            'Average Reading Score' : average_reading_score,
            '% Passing Math' : passing_rate_math,
            '% Passing Reading' : passing_rate_reading,
            '% Overall Passing Rate' : overall_passing_rate
        },
    index = [0]
)

district_summary_df['Total Budget'] = district_summary_df['Total Budget'].map('$ {:,.2f}'.format)

district_summary_df = district_summary_df[['Total Schools', 'Total Students', 'Total Budget', 
                                           'Average Math Score', 'Average Reading Score', 
                                           '% Passing Math', '% Passing Reading', '% Overall Passing Rate']]

district_summary_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Total Schools</th>
      <th>Total Students</th>
      <th>Total Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15</td>
      <td>39170</td>
      <td>$ 24,649,428.00</td>
      <td>78.985371</td>
      <td>81.87784</td>
      <td>90.906306</td>
      <td>100.0</td>
      <td>95.453153</td>
    </tr>
  </tbody>
</table>
</div>



<big><b>School Summary</b></big>


```python
total_students_school = students_df.groupby('school')['name'].count()
total_budget_school = schools_df.groupby('name')['budget'].sum()

avg_math_score_school = students_df.groupby('school')['math_score'].sum() / total_students_school
avg_reading_score_school = students_df.groupby('school')['reading_score'].sum() / total_students_school

passing_math_school = students_df.loc[students_df['math_score'] > 60, :].groupby('school')['math_score'].count()
passing_reading_school = students_df.loc[students_df['reading_score'] > 60, :].groupby('school')['reading_score'].count()

schools_df_addendum = pd.concat([total_students_school, 
                                 total_budget_school, 
                                 avg_math_score_school, 
                                 avg_reading_score_school, 
                                 passing_math_school, 
                                 passing_reading_school], 
                               axis = 1)

schools_df_addendum.reset_index(inplace=True)

schools_df_addendum = schools_df_addendum.rename(columns={'index' : 'name',  
                                                        'name' : 'Total Students',
                                                        'budget' : 'Total School Budget',
                                                        0 : 'Average Math Score',
                                                        1 : 'Average Reading Score',
                                                        'math_score' : 'passing math raw',
                                                        'reading_score' : 'passing reading raw'
                                                        })

schools_df_addendum = schools_df.merge(schools_df_addendum, on='name')

schools_df_addendum['Per Student Budget'] = schools_df_addendum['Total School Budget'] / schools_df_addendum['Total Students']
schools_df_addendum['% Passing Math'] = (schools_df_addendum['passing math raw'] / schools_df_addendum['Total Students']) * 100
schools_df_addendum['% Passing Reading'] = (schools_df_addendum['passing reading raw'] / schools_df_addendum['Total Students']) * 100
schools_df_addendum['% Overall Passing Rate'] = (schools_df_addendum['% Passing Math'] + schools_df_addendum['% Passing Reading']) / 2

schools_summary = schools_df_addendum[['name', 
                                       'type', 
                                       'Total Students', 
                                       'Total School Budget', 
                                       'Per Student Budget', 
                                       'Average Math Score', 
                                       'Average Reading Score', 
                                       '% Passing Math', 
                                       '% Passing Reading', 
                                       '% Overall Passing Rate']]
                                       
schools_summary = schools_summary.rename(columns={'type' : 'School Type'})
schools_summary.set_index('name', inplace=True)
del schools_summary.index.name

schools_summary['Total School Budget'] = schools_summary['Total School Budget'].map('$ {:,.2f}'.format)
schools_summary['Per Student Budget'] = schools_summary['Per Student Budget'].map('$ {:,.2f}'.format)


schools_summary

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>2917</td>
      <td>$ 1,910,635.00</td>
      <td>$ 655.00</td>
      <td>76.629414</td>
      <td>81.182722</td>
      <td>86.835790</td>
      <td>100.0</td>
      <td>93.417895</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>2949</td>
      <td>$ 1,884,411.00</td>
      <td>$ 639.00</td>
      <td>76.711767</td>
      <td>81.158020</td>
      <td>86.436080</td>
      <td>100.0</td>
      <td>93.218040</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>Charter</td>
      <td>1761</td>
      <td>$ 1,056,600.00</td>
      <td>$ 600.00</td>
      <td>83.359455</td>
      <td>83.725724</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>District</td>
      <td>4635</td>
      <td>$ 3,022,020.00</td>
      <td>$ 652.00</td>
      <td>77.289752</td>
      <td>80.934412</td>
      <td>86.450917</td>
      <td>100.0</td>
      <td>93.225458</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>Charter</td>
      <td>1468</td>
      <td>$ 917,500.00</td>
      <td>$ 625.00</td>
      <td>83.351499</td>
      <td>83.816757</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>2283</td>
      <td>$ 1,319,574.00</td>
      <td>$ 578.00</td>
      <td>83.274201</td>
      <td>83.989488</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>Charter</td>
      <td>1858</td>
      <td>$ 1,081,356.00</td>
      <td>$ 582.00</td>
      <td>83.061895</td>
      <td>83.975780</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>Bailey High School</th>
      <td>District</td>
      <td>4976</td>
      <td>$ 3,124,928.00</td>
      <td>$ 628.00</td>
      <td>77.048432</td>
      <td>81.033963</td>
      <td>87.439711</td>
      <td>100.0</td>
      <td>93.719855</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>Charter</td>
      <td>427</td>
      <td>$ 248,087.00</td>
      <td>$ 581.00</td>
      <td>83.803279</td>
      <td>83.814988</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>962</td>
      <td>$ 585,858.00</td>
      <td>$ 609.00</td>
      <td>83.839917</td>
      <td>84.044699</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>Charter</td>
      <td>1800</td>
      <td>$ 1,049,400.00</td>
      <td>$ 583.00</td>
      <td>83.682222</td>
      <td>83.955000</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>3999</td>
      <td>$ 2,547,363.00</td>
      <td>$ 637.00</td>
      <td>76.842711</td>
      <td>80.744686</td>
      <td>86.446612</td>
      <td>100.0</td>
      <td>93.223306</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>4761</td>
      <td>$ 3,094,650.00</td>
      <td>$ 650.00</td>
      <td>77.072464</td>
      <td>80.966394</td>
      <td>86.704474</td>
      <td>100.0</td>
      <td>93.352237</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>District</td>
      <td>2739</td>
      <td>$ 1,763,916.00</td>
      <td>$ 644.00</td>
      <td>77.102592</td>
      <td>80.746258</td>
      <td>87.221614</td>
      <td>100.0</td>
      <td>93.610807</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>Charter</td>
      <td>1635</td>
      <td>$ 1,043,130.00</td>
      <td>$ 638.00</td>
      <td>83.418349</td>
      <td>83.848930</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
  </tbody>
</table>
</div>



<big><b>Top Performing Schools (By Passing Rate)</b></big>


```python
#top_performing_schools = schools_summary.sort_values(['% Overall Passing Rate', 'Per Student Budget'], ascending=[False, True]).iloc[0:5,]
top_performing_schools = schools_summary.sort_values('% Overall Passing Rate', ascending=False).iloc[0:5,]
top_performing_schools
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Shelton High School</th>
      <td>Charter</td>
      <td>1761</td>
      <td>$ 1,056,600.00</td>
      <td>$ 600.00</td>
      <td>83.359455</td>
      <td>83.725724</td>
      <td>100.0</td>
      <td>100.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>Charter</td>
      <td>1468</td>
      <td>$ 917,500.00</td>
      <td>$ 625.00</td>
      <td>83.351499</td>
      <td>83.816757</td>
      <td>100.0</td>
      <td>100.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>2283</td>
      <td>$ 1,319,574.00</td>
      <td>$ 578.00</td>
      <td>83.274201</td>
      <td>83.989488</td>
      <td>100.0</td>
      <td>100.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>Charter</td>
      <td>1858</td>
      <td>$ 1,081,356.00</td>
      <td>$ 582.00</td>
      <td>83.061895</td>
      <td>83.975780</td>
      <td>100.0</td>
      <td>100.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>Charter</td>
      <td>427</td>
      <td>$ 248,087.00</td>
      <td>$ 581.00</td>
      <td>83.803279</td>
      <td>83.814988</td>
      <td>100.0</td>
      <td>100.0</td>
      <td>100.0</td>
    </tr>
  </tbody>
</table>
</div>



<big><b>Bottom Performing Schools (By Passing Rate)</b></big>


```python
#bottom_performing_schools = schools_summary.sort_values(['% Overall Passing Rate', 'Per Student Budget'], ascending=[True, False]).iloc[0:5,]
bottom_performing_schools = schools_summary.sort_values('% Overall Passing Rate').iloc[0:5,]
bottom_performing_schools
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>2949</td>
      <td>$ 1,884,411.00</td>
      <td>$ 639.00</td>
      <td>76.711767</td>
      <td>81.158020</td>
      <td>86.436080</td>
      <td>100.0</td>
      <td>93.218040</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>3999</td>
      <td>$ 2,547,363.00</td>
      <td>$ 637.00</td>
      <td>76.842711</td>
      <td>80.744686</td>
      <td>86.446612</td>
      <td>100.0</td>
      <td>93.223306</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>District</td>
      <td>4635</td>
      <td>$ 3,022,020.00</td>
      <td>$ 652.00</td>
      <td>77.289752</td>
      <td>80.934412</td>
      <td>86.450917</td>
      <td>100.0</td>
      <td>93.225458</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>4761</td>
      <td>$ 3,094,650.00</td>
      <td>$ 650.00</td>
      <td>77.072464</td>
      <td>80.966394</td>
      <td>86.704474</td>
      <td>100.0</td>
      <td>93.352237</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>2917</td>
      <td>$ 1,910,635.00</td>
      <td>$ 655.00</td>
      <td>76.629414</td>
      <td>81.182722</td>
      <td>86.835790</td>
      <td>100.0</td>
      <td>93.417895</td>
    </tr>
  </tbody>
</table>
</div>



<big><b>Math Scores by Grade</b></big>


```python
students_9  = students_df.loc[students_df['grade'] == '9th',]
students_10 = students_df.loc[students_df['grade'] == '10th',]
students_11 = students_df.loc[students_df['grade'] == '11th',]
students_12 = students_df.loc[students_df['grade'] == '12th',]

students_9_school_count  = students_9.groupby('school')['grade'].count()
students_10_school_count = students_10.groupby('school')['grade'].count()
students_11_school_count = students_11.groupby('school')['grade'].count()
students_12_school_count = students_12.groupby('school')['grade'].count()

math_9_school  = students_9.groupby('school')['math_score'].sum() / students_9_school_count
math_10_school = students_10.groupby('school')['math_score'].sum() / students_10_school_count
math_11_school = students_11.groupby('school')['math_score'].sum() / students_11_school_count
math_12_school = students_12.groupby('school')['math_score'].sum() / students_12_school_count

math_scores_grade_school = pd.concat([math_9_school, 
                                    math_10_school,
                                    math_11_school,
                                    math_12_school],
                                    axis = 1)

math_scores_grade_school = math_scores_grade_school.rename(columns={0 :'9th', 1 : '10th', 2 : '11', 3 : '12'})
del math_scores_grade_school.index.name

math_scores_grade_school

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>9th</th>
      <th>10th</th>
      <th>11</th>
      <th>12</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>77.083676</td>
      <td>76.996772</td>
      <td>77.515588</td>
      <td>76.492218</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>83.094697</td>
      <td>83.154506</td>
      <td>82.765560</td>
      <td>83.277487</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>76.403037</td>
      <td>76.539974</td>
      <td>76.884344</td>
      <td>77.151369</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>77.361345</td>
      <td>77.672316</td>
      <td>76.918058</td>
      <td>76.179963</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>82.044010</td>
      <td>84.229064</td>
      <td>83.842105</td>
      <td>83.356164</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>77.438495</td>
      <td>77.337408</td>
      <td>77.136029</td>
      <td>77.186567</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.787402</td>
      <td>83.429825</td>
      <td>85.000000</td>
      <td>82.855422</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>77.027251</td>
      <td>75.908735</td>
      <td>76.446602</td>
      <td>77.225641</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>77.187857</td>
      <td>76.691117</td>
      <td>77.491653</td>
      <td>76.863248</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.625455</td>
      <td>83.372000</td>
      <td>84.328125</td>
      <td>84.121547</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>76.859966</td>
      <td>76.612500</td>
      <td>76.395626</td>
      <td>77.690748</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>83.420755</td>
      <td>82.917411</td>
      <td>83.383495</td>
      <td>83.778976</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.590022</td>
      <td>83.087886</td>
      <td>83.498795</td>
      <td>83.497041</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.085578</td>
      <td>83.724422</td>
      <td>83.195326</td>
      <td>83.035794</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>83.264706</td>
      <td>84.010288</td>
      <td>83.836782</td>
      <td>83.644986</td>
    </tr>
  </tbody>
</table>
</div>



<big><b></b></big>

<big><b>Reading Score by Grade</b></big>


```python
reading_9_school = students_9.groupby('school')['reading_score'].sum() / students_9_school_count
reading_10_school = students_10.groupby('school')['reading_score'].sum() / students_10_school_count
reading_11_school = students_11.groupby('school')['reading_score'].sum() / students_11_school_count
reading_12_school = students_12.groupby('school')['reading_score'].sum() / students_12_school_count

reading_scores_grade_school = pd.concat([reading_9_school, 
                                    reading_10_school,
                                    reading_11_school,
                                    reading_12_school],
                                    axis = 1)

reading_scores_grade_school = reading_scores_grade_school.rename(columns={0 :'9th', 1 : '10th', 2 : '11', 3 : '12'})
del reading_scores_grade_school.index.name

reading_scores_grade_school

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>9th</th>
      <th>10th</th>
      <th>11</th>
      <th>12</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>81.303155</td>
      <td>80.907183</td>
      <td>80.945643</td>
      <td>80.912451</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>83.676136</td>
      <td>84.253219</td>
      <td>83.788382</td>
      <td>84.287958</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>81.198598</td>
      <td>81.408912</td>
      <td>80.640339</td>
      <td>81.384863</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>80.632653</td>
      <td>81.262712</td>
      <td>80.403642</td>
      <td>80.662338</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>83.369193</td>
      <td>83.706897</td>
      <td>84.288089</td>
      <td>84.013699</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>80.866860</td>
      <td>80.660147</td>
      <td>81.396140</td>
      <td>80.857143</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.677165</td>
      <td>83.324561</td>
      <td>83.815534</td>
      <td>84.698795</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>81.290284</td>
      <td>81.512386</td>
      <td>81.417476</td>
      <td>80.305983</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>81.260714</td>
      <td>80.773431</td>
      <td>80.616027</td>
      <td>81.227564</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.807273</td>
      <td>83.612000</td>
      <td>84.335938</td>
      <td>84.591160</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>80.993127</td>
      <td>80.629808</td>
      <td>80.864811</td>
      <td>80.376426</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>84.122642</td>
      <td>83.441964</td>
      <td>84.373786</td>
      <td>82.781671</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.728850</td>
      <td>84.254157</td>
      <td>83.585542</td>
      <td>83.831361</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.939778</td>
      <td>84.021452</td>
      <td>83.764608</td>
      <td>84.317673</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>83.833333</td>
      <td>83.812757</td>
      <td>84.156322</td>
      <td>84.073171</td>
    </tr>
  </tbody>
</table>
</div>



<big><b>Scores by School Spending</b></big>


```python
spending_df = schools_df
spending_students_df = students_df

spending_df['Per Student Budget'] = spending_df['budget'] / spending_df['size']

bins = [0, 585, 615, 645, 675]
group_labels = ['<$585','$585-615','$615-645','$645-675']
spending_df['Spending Ranges (Per Student)'] = pd.cut(spending_df['Per Student Budget'],bins,labels=group_labels)

spending_students_df = spending_students_df.merge(spending_df, left_on='school', right_on='name')

students_by_budget = spending_students_df.groupby('Spending Ranges (Per Student)')['name_x'].count()

avg_math_score_budget = spending_students_df.groupby('Spending Ranges (Per Student)')['math_score'].sum() / students_by_budget
avg_reading_score_budget = spending_students_df.groupby('Spending Ranges (Per Student)')['reading_score'].sum() / students_by_budget

passing_math_budget = (spending_students_df.loc[spending_students_df['math_score'] > 60, :].groupby('Spending Ranges (Per Student)')['math_score'].count() / students_by_budget) * 100
passing_reading_budget = (spending_students_df.loc[spending_students_df['reading_score'] > 60, :].groupby('Spending Ranges (Per Student)')['reading_score'].count() / students_by_budget) * 100

scores_by_spending = pd.concat([avg_math_score_budget, 
                                 avg_reading_score_budget, 
                                 passing_math_budget, 
                                 passing_reading_budget], 
                               axis = 1)

scores_by_spending = scores_by_spending.rename(columns={0 : 'Average Math Score',
                                                       1 : 'Average Reading Score',
                                                       2 : '% Passing Math',
                                                       3 : '% Passing Reading'})

scores_by_spending['% Overall Passing Rate'] = (scores_by_spending['% Passing Math'] + scores_by_spending['% Passing Reading']) / 2

scores_by_spending.index.rename('Spending Ranges (Per Student)', inplace=True)

scores_by_spending
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>Spending Ranges (Per Student)</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>&lt;$585</th>
      <td>83.363065</td>
      <td>83.964039</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>$585-615</th>
      <td>83.529196</td>
      <td>83.838414</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>$615-645</th>
      <td>78.061635</td>
      <td>81.434088</td>
      <td>89.209726</td>
      <td>100.0</td>
      <td>94.604863</td>
    </tr>
    <tr>
      <th>$645-675</th>
      <td>77.049297</td>
      <td>81.005604</td>
      <td>86.640136</td>
      <td>100.0</td>
      <td>93.320068</td>
    </tr>
  </tbody>
</table>
</div>



<big><b>Scores by School Size</b></big>


```python
size_df = schools_df
size_students_df = students_df

size_df['Per Student Budget'] = size_df['budget'] / size_df['size']

bins = [0, 1000, 2000, 5000]
group_labels = ['Small (<1000)','Medium (1000-2000)','Large (2000-5000)']
size_df['School Sizes'] = pd.cut(size_df['size'],bins,labels=group_labels)

size_students_df = size_students_df.merge(size_df, left_on='school', right_on='name')

size_students_df.drop(['name_y'], axis=1, inplace=True)

students_by_size = size_students_df.groupby('School Sizes')['name_x'].count()

avg_math_score_size = size_students_df.groupby('School Sizes')['math_score'].sum() / students_by_size
avg_reading_score_size = size_students_df.groupby('School Sizes')['reading_score'].sum() / students_by_size

passing_math_size = (size_students_df.loc[size_students_df['math_score'] > 60, :].groupby('School Sizes')['math_score'].count() / students_by_size) * 100
passing_reading_size = (size_students_df.loc[size_students_df['reading_score'] > 60, :].groupby('School Sizes')['reading_score'].count() / students_by_size) * 100


scores_by_size = pd.concat([avg_math_score_size, 
                                 avg_reading_score_size, 
                                 passing_math_size, 
                                 passing_reading_size], 
                               axis = 1)

scores_by_size = scores_by_size.rename(columns={0 : 'Average Math Score',
                                                       1 : 'Average Reading Score',
                                                       2 : '% Passing Math',
                                                       3 : '% Passing Reading'})

scores_by_size['% Overall Passing Rate'] = (scores_by_size['% Passing Math'] + scores_by_size['% Passing Reading']) / 2

scores_by_size.index.rename('School Sizes', inplace=True)

scores_by_size

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>School Sizes</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Small (&lt;1000)</th>
      <td>83.828654</td>
      <td>83.974082</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>Medium (1000-2000)</th>
      <td>83.372682</td>
      <td>83.867989</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>Large (2000-5000)</th>
      <td>77.477597</td>
      <td>81.198674</td>
      <td>87.825968</td>
      <td>100.0</td>
      <td>93.912984</td>
    </tr>
  </tbody>
</table>
</div>



<big><b>Scores by School Type</b></big>


```python
type_students_df = size_students_df

students_by_type = type_students_df.groupby('type')['name_x'].count()

avg_math_score_type = type_students_df.groupby('type')['math_score'].sum() / students_by_type
avg_reading_score_type = type_students_df.groupby('type')['reading_score'].sum() / students_by_type

passing_math_type = (type_students_df.loc[type_students_df['math_score'] > 60, :].groupby('type')['math_score'].count() / students_by_type) * 100
passing_reading_type = (type_students_df.loc[type_students_df['reading_score'] > 60, :].groupby('type')['reading_score'].count() / students_by_type) * 100


scores_by_type = pd.concat([avg_math_score_type, 
                                 avg_reading_score_type, 
                                 passing_math_type, 
                                 passing_reading_type], 
                               axis = 1)

scores_by_type = scores_by_type.rename(columns={0 : 'Average Math Score',
                                                       1 : 'Average Reading Score',
                                                       2 : '% Passing Math',
                                                       3 : '% Passing Reading'})

scores_by_type['% Overall Passing Rate'] = (scores_by_type['% Passing Math'] + scores_by_type['% Passing Reading']) / 2

scores_by_type.index.rename('School Type', inplace=True)

scores_by_type
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>School Type</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Charter</th>
      <td>83.406183</td>
      <td>83.902821</td>
      <td>100.00000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>District</th>
      <td>76.987026</td>
      <td>80.962485</td>
      <td>86.79567</td>
      <td>100.0</td>
      <td>93.397835</td>
    </tr>
  </tbody>
</table>
</div>


