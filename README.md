House Prices - Advanced Regression Techniques
კონკურსის მიმოხილვა

Kaggle-ის House Prices კონკურსის მიზანია საცხოვრებელი სახლების ფასის პროგნოზირება სხვადასხვა მახასიათებლის საფუძველზე (მაგ: ფართობი, მდებარეობა, ხარისხი და ა.შ.).

მოდელი ფასდება Root Mean Squared Error (RMSE) მეტრიკით ლოგარითმულ სკალაზე (log RMSE).

რეპოზიტორიის სტრუქტურა
proj1/
│
├── experiment.ipynb ← EDA, preprocessing, feature engineering, ექსპერიმენტები
├── inference.ipynb ← საუკეთესო მოდელით prediction და submission
├── README.md
ფაილების აღწერა
ფაილი აღწერა
experiment.ipynb მონაცემთა ანალიზი, preprocessing, feature engineering და მოდელების ექსპერიმენტები
inference.ipynb საბოლოო მოდელის გამოყენება test მონაცემებზე და submission გენერაცია
მონაცემთა გაწმენდა და დამუშავება (Cleaning & Preprocessing)

1. Missing values ანალიზი

მონაცემებში გამოვთვალე თითოეული სვეტის missing value პროცენტი:

na_percent = (train_df.isnull().sum() / len(train_df)) \* 100

![Alt text](imageNAs.png)

2. მაღალი missing value მქონე სვეტების მოცილება

სვეტები, სადაც missing value ≥ 40% იყო, სრულად ამოვიღე.

მიზეზი: როდესაც სვეტის დიდი ნაწილი ცარიელია, მისი შევსება ხშირად noise-ს ამატებს და მოდელს აუარესებს.

3. კატეგორიული სვეტების კოდირება

კატეგორიული სვეტები გადავიყვანე რიცხვებად factorize მეთოდით.

ეს ნიშნავს, რომ თითოეულ კატეგორიას მიენიჭა უნიკალური integer მნიშვნელობა.

4. რიცხვითი NA მნიშვნელობების შევსება

შემდეგ სვეტებში:

LotFrontage
GarageYrBlt
MasVnrArea

გამოვიყენე მედიანა:

median = X_train[col].median()

მედიანა ავირჩიე იმიტომ, რომ ის უფრო მდგრადია outliers-ის მიმართ.

Feature Engineering

1. Target transformation
   y = np.log1p(train_df["SalePrice"])

მიზეზი:

ფასები skewed არის
კონკურსი იყენებს log RMSE-ს
→ ეს მნიშვნელოვნად აუმჯობესებს შედეგს

2.  Skewed feature-ების ტრანსფორმაცია

ვიპოვე skewed სვეტები და გამოვიყენე log ტრანსფორმაცია:

skewed_feats = X_train[numeric_feats].skew()
np.log1p()

მიზანი:

დიდი მნიშვნელობების შემცირება
outlier-ების გავლენის შემცირება
უკეთესი განაწილება
Feature Selection

კორელაციის ფილტრი (Threshold = 0.8)

ამოვიღე სვეტები, რომლებიც ერთმანეთთან ძალიან ძლიერად იყო დაკავშირებული.

მიზეზი:

მაღალი კორელაცია ნიშნავს redundant ინფორმაციას
ამცირებს overfitting-ს

კატეგორიული Feature-ების გაუმჯობესება

1. Ordinal encoding (quality features)

შემდეგ სვეტებში გამოვიყენე ranking:

ExterQual
BsmtQual
KitchenQual
GarageQual
{"Ex":5, "Gd":4, "TA":3, "Fa":2, "Po":1}

ამ სვეტებს აქვთ ბუნებრივი რიგი, ამიტომ ასეთი კოდირება უკეთ მუშაობს.

2. Missing კატეგორიების დამუშავება
   fillna("None")

ეს ნიშნავს, რომ "None" ხდება ცალკე კატეგორია.

3. One-Hot Encoding

ნომინალური კატეგორიები გადავიყვანე:

pd.get_dummies()

შემდეგ დავასინქრონე train, validation და test მონაცემები:

align()

მოდელი

გამოვიყენე XGBoost Regressor:

xgb.XGBRegressor(
n_estimators=1000,
max_depth=5,
learning_rate=0.05,
subsample=0.8,
colsample_bytree=0.8
)

რატომ XGBoost:

კარგად მუშაობს tabular მონაცემებზე
ავტომატურად არჩევს მნიშვნელოვან feature-ებს
robust არის noise-ის მიმართ
შეფასება

Validation-ზე გამოვითვალე RMSE:

np.sqrt(mean_squared_error(y_val, preds))
Submission

რადგან target log-ტრანსფორმირებულია:

test_preds = np.expm1(model.predict(test_df))

შედეგი

Kaggle Public Score: ~0.135
