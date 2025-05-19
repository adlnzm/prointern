# **Customer Churn Prediction: Documentation**

This documentation explains the Python script used to build a **machine learning model** to predict customer churn in a telecom company. The model is trained using the **Random Forest Classifier** from the `scikit-learn` library.


## **1. Import Required Libraries**

```python
import pandas as pd
from sklearn.preprocessing import LabelEncoder
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import classification_report, accuracy_score
```

* `pandas`: For loading and manipulating datasets.
* `LabelEncoder`: To convert text labels into numbers.
* `train_test_split`: To split data into training and test sets.
* `RandomForestClassifier`: Our machine learning model.
* `classification_report`, `accuracy_score`: For evaluating model performance.


## **2. Load the Datasets**

```python
churn_df = pd.read_csv("telecom_customer_churn.csv")
zipcode_df = pd.read_csv("telecom_zipcode_population.csv")
```

* `churn_df`: Main customer churn dataset.
* `zipcode_df`: Contains population info for zip codes.


## **3. Merge Population Data**

```python
churn_df['Zip Code'] = churn_df['Zip Code'].astype(str).str.zfill(5)
zipcode_df['Zip Code'] = zipcode_df['Zip Code'].astype(str).str.zfill(5)
merged_df = pd.merge(churn_df, zipcode_df, on='Zip Code', how='left')
```

* Ensure zip codes are strings with 5 digits.
* Merge the population info into the main dataset.


## **4. Handle Missing Data**

```python
# Fill missing service fields
service_features = ['Online Backup', ..., 'Streaming Movies']
merged_df[service_features] = merged_df[service_features].fillna('Unknown')

# Fill numerical fields
merged_df['Avg Monthly GB Download'] = merged_df['Avg Monthly GB Download'].fillna(0)
merged_df['Avg Monthly Long Distance Charges'] = merged_df['Avg Monthly Long Distance Charges'].fillna(0)
merged_df['Multiple Lines'] = merged_df['Multiple Lines'].fillna('Unknown')
```

* Missing values in service columns are filled with `"Unknown"`.
* Missing numbers are filled with `0`.


## **5. Drop Unnecessary Columns**

```python
drop_columns = ['Customer ID', 'Churn Reason', 'Churn Category', 'City', 'Latitude', 'Longitude']
clean_df = merged_df.drop(columns=drop_columns)
```

These columns are either identifiers or not useful for prediction.


## **6. Create Target Variable (Churn)**

```python
clean_df = clean_df.dropna(subset=['Customer Status'])
clean_df['Churn'] = clean_df['Customer Status'].apply(lambda x: 1 if x == 'Churned' else 0)
clean_df = clean_df.drop(columns=['Customer Status'])
```

* Converts the text label `"Churned"` to `1` and others to `0`.


## **7. Encode Categorical Variables**

```python
for column in clean_df.select_dtypes(include='object').columns:
    le = LabelEncoder()
    clean_df[column] = le.fit_transform(clean_df[column])
```

* Text values (like "Yes", "No") are converted to numbers for the model.


## **8. Split Data for Training and Testing**

```python
X = clean_df.drop(columns=['Churn'])
y = clean_df['Churn']
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
```

* `X`: Features used to predict churn.
* `y`: Target (whether the customer churned).
* 80% used for training, 20% for testing.


## **9. Train the Random Forest Model**

```python
model = RandomForestClassifier(random_state=42)
model.fit(X_train, y_train)
```

* A Random Forest model is trained on the data.


## **10. Evaluate Model Accuracy**

```python
y_pred = model.predict(X_test)
accuracy = accuracy_score(y_test, y_pred)
report = classification_report(y_test, y_pred)

print("Model Accuracy: {:.2f}%".format(accuracy * 100))
print("\nClassification Report:\n", report)
```

* Accuracy is printed (percentage of correct predictions).
* `classification_report` gives details on precision, recall, and F1-score.


## **Final Output Example**

```
Model Accuracy: 83.53%

Classification Report:
              precision    recall  f1-score   support

           0       0.86      0.93      0.89      1036
           1       0.74      0.58      0.65       373

    accuracy                           0.84      1409
   macro avg       0.80      0.75      0.77      1409
weighted avg       0.83      0.84      0.83      1409
```

## Summary

* This project helps predict if a telecom customer will **churn**.
* We clean, prepare, and train a model using real customer data.
* The model achieves **83% accuracy** — useful for real business decisions.


