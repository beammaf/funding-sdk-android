### Funding SDK - Android Guide

The Funding SDK is for interacting with the Beam Wallet platform to add, update or remove vehicles for integrated Ticketless Parking System. 

In order to use SDK you must be a registered developer with a provisioned API key.

## Requirements
* SDK Supports min api level 18.
* SDK Requires Java 8.
* BAPFES Authentication token. 

## Integration
* Java 8+ support should be added to project. To do that below code should be added to project level gradle file.
  ```groovy
  android {
      compileOptions{
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
  }
  ```
* SDK bundle is an .aar file. This aar file should be added as dependency to project level gradle file.
  ```groovy
  dependencies {
       compile(name:'funding-sdk', ext:'aar')
  }
  ```

* SDK Also requires some extra libraries (RX Java,Retrofit)

```groovy

        RX_VERSION = '2.1.4'
        RETROFIT_VERSION = '2.4.0'



 
        implementation "io.reactivex.rxjava2:rxjava:$RX_VERSION"
        implementation "io.reactivex.rxjava2:rxandroid:2.0.2"
    
        /* Networking */
        implementation "com.squareup.retrofit2:retrofit:$RETROFIT_VERSION"
        implementation "com.squareup.retrofit2:converter-gson:$RETROFIT_VERSION"
        implementation "com.google.code.gson:gson:2.8.2"
        implementation "com.squareup.retrofit2:adapter-rxjava2:$RETROFIT_VERSION"
        implementation 'com.github.rtyley:spongycastle:sc-v1.54.0.0'


 ```

# SDK Overview

## Funding TokenProvider

 * FSTokenProvider is an interface in order to provide authentication token from BAPFES. This interface should be implemented to feed sdk with token.
 * Make sure to provide bapfes token before calling vehicle actions.
 ```java
 new FSTokenProvider() {
                     @Override
                     public String getToken() {
                        return <<BAPFES TOKEN>>
                     }
                 }
 ```

## Initialize and Start Sdk
* In order to in initialize sdk, FSSDK start() function should called from Instance

```java
  void start(@NonNull FSServer server, @NonNull FSTokenProvider tokenProvider, FSInitializationListener listener);
```
* FSServer: This is an enum in order to specify environment of funding service.



Example Usage: 
```java

FSSdk.getInstance().start(FSServer.STAGING, this, new FSInitializationListener() {
            @Override
            public void onSdkInitialized() {
                sdkInitialized = true;
                if (view.isAvailable()) {
                    view.showInfo("Sdk initialized successfully");
                    view.hideProgress();
                }

            }

            @Override
            public void onError(FSError error) {
                if (view.isAvailable()) {
                    view.hideProgress();
                    view.showError(error.getMessage());
                }
            }
        });

```

## FSSDK 
* All sdk functions can be accessed from FSSDK object. It's a simple interface to access functionalities of Funding SDK

```java
  public interface FSSdk {
  
    static FSSdk getInstance() {
        return FSSdkImpl.getInstance();
    }


  void start(@NonNull FSServer server, @NonNull FSTokenProvider tokenProvider, FSInitializationListener listener);

  void getFundingSources(FSCallback<List<CreditCard>> callback);

  void addCreditCard(Activity activity, int requestCode);

  void addCreditCard(Activity activity, int requestCode, ActivityOptions options);

  void verifyCard(CreditCard fundingSource, Activity activity, int requestCode);

  void verifyCard(CreditCard fundingSource, Activity activity, int requestCode, ActivityOptions options);

  void removeCreditCard(CreditCard fundingSources, FSCallback<Boolean> listener);


  void clear();
  }

```

#### FSCallback\<T>
* An interface in order to get results for async operations.

#### FSServer
* An enum object to specify target environment of sdk
```java
public enum FSServer {
    DEV,
    PRODUCTION,
    STAGING
}

```


###  Credit Card Operations
#### CreditCard
* This class is a model used as a refference of CreditCard
```java
public class CreditCard implements Serializable{

    private String uuid;
    private String cardNumber;
    private String cvv;
    private String expiry;
    private String nameSurname;
    private boolean requiresVerification = false;
    private CardStatus statusName;
}
    
```

#### Get Credit Cards
```java
 FSSdk.getInstance().getFundingSources(new FSCallback<List<CreditCard>>() {
            @Override
            public void onSuccess(List<CreditCard> result) {
                
            }

            @Override
            public void onError(FSError error) {

            }
        });
```
* This function returns registered credit cards which associated to logged in user.
#### Add Credit Card
In order to add credit card, there is a function called addCreditCard(). This function is going to launch.  addCreditCardActivity which is inside the Funding  SDK.
This UI is fully customizable

For more information about customizing UI, Please check "Customizing UI" section.

```java
  FSSdk.getInstance().addCreditCard(view.getActivity(), ADD_CARD_REQUEST_CODE)
```

This function launch an activity for result. 

```java
public void activityResult(int requestCode, int resultCode, Intent data) {
        if (requestCode == ADD_CARD_REQUEST_CODE) {
            if (resultCode == Activity.RESULT_OK) {
                if (data != null) {
                    CreditCard card = (CreditCard) data.getSerializableExtra(FSSdk.BUNDLE_CARD_OPERATION_RESULT);
                    if (card != null) {
                        dataSet.add(card);
                    }
                }
            }
        }
}
```

### Verify Credit Card
In order to verify credit card, there is a function called verifyCreditCard(). This function is going to launch verifyCreditCardActivity which is inside the Funding SDK.

This UI is fully customizable

For more information about customizing UI, Please check "Customizing UI" section.

```java
 FSSdk.getInstance().verifyCard(event.getCreditCard(), view.getActivity(), VERIFY_CARD_REQUEST_CODE);

```

This function launch an activity for result. 

```java
public void activityResult(int requestCode, int resultCode, Intent data) {
        if (requestCode == VERIFY_CARD_REQUEST_CODE) {
            if (resultCode == Activity.RESULT_OK) {
                if (data != null) {
                    CreditCard card = (CreditCard) data.getSerializableExtra(FSSdk.BUNDLE_CARD_OPERATION_RESULT);
                }
            }
        }
}
```


## UI Customization.
TPS SDK supports custom UI for AddCreditCard and VerifyCreditCard functionalities. 


### Add Credit Card Page
In order to customize Add Credit card ui, **tps_add_card_layout.xml** should be added to project. TPS SDK automatically detects if tps_add_card_layout.xml exists and render it to screen. This xml should contain this fields with specified ids.

|  View Type   | Description              |id              |
| ------------ | ------------------------ |--------------- |
|  EditText | Credit Card Number Field |edtPanTps       |
|  EditText | Name Surname             |edtFulltNameTps |
|  EditText | CVC Number               |edtCvcTps       |
|  EditText | Expiry Date Field        |edtExpiryTps    |
|  Button   | Submit Button            |btnSubmitTps    |



#### Example Layout
 
```xml
<?xml version="1.0" encoding="utf-8"?>

<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:support="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <android.support.v7.widget.Toolbar
        android:id="@+id/viewToolbar"
        android:layout_width="0dip"
        android:layout_height="wrap_content"
        android:background="@color/colorPrimary"
        android:minHeight="?android:attr/actionBarSize"
        android:theme="@style/ThemeOverlay.AppCompat.Light"
        support:layout_constraintEnd_toEndOf="parent"
        support:layout_constraintStart_toStartOf="parent"
        support:layout_constraintTop_toTopOf="parent"
        support:navigationIcon="@drawable/ic_arrow_back"
        support:title="@string/str_add_new_card"
        support:titleTextColor="@color/colorToolbarTextAndIconTint" />

    <android.support.v7.widget.AppCompatImageView
        android:id="@+id/imgCard"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginStart="@dimen/spacing_2x"
        android:alpha="0.6"
        android:src="@drawable/ic_credit_card_black_24dp"
        support:layout_constraintBottom_toBottomOf="@id/number_layout"
        support:layout_constraintStart_toStartOf="parent"
        support:layout_constraintTop_toTopOf="@id/number_layout"

        tools:layout_editor_absoluteX="0dp"
        tools:layout_editor_absoluteY="72dp" />

    <android.support.design.widget.TextInputLayout
        android:id="@+id/number_layout"
        style="@style/Beam.TextInputLayout"
        android:layout_width="0dip"
        android:layout_height="wrap_content"
        android:layout_marginBottom="@dimen/spacing_2x"
        android:layout_marginEnd="@dimen/spacing_2x"
        android:layout_marginStart="@dimen/spacing_2x"
        android:layout_marginTop="@dimen/spacing_2x"
        android:theme="@style/Beam.TextInputLayout"
        support:layout_constraintEnd_toEndOf="parent"
        support:layout_constraintStart_toEndOf="@id/imgCard"
        support:layout_constraintTop_toBottomOf="@+id/viewToolbar">

        <android.support.design.widget.TextInputEditText
            android:id="@+id/edtPanTps"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:digits="0,1,2,3,4,5,6,7,8,9, "
            android:drawableEnd="@drawable/icon_card_mastercard"
            android:hint="@string/label_card_number"
            android:inputType="number"
            android:maxLength="19" />
    </android.support.design.widget.TextInputLayout>


    <android.support.design.widget.TextInputLayout
        android:id="@+id/expiry_layout"
        style="@style/Beam.TextInputLayout"
        android:layout_width="0dip"
        android:layout_height="wrap_content"
        android:layout_marginBottom="@dimen/spacing_2x"
        android:layout_marginEnd="@dimen/spacing_2x"
        android:layout_marginTop="@dimen/spacing_2x"
        android:theme="@style/Beam.TextInputLayout"
        support:layout_constraintEnd_toStartOf="@id/cvc_layout"
        support:layout_constraintStart_toStartOf="@id/number_layout"
        support:layout_constraintTop_toBottomOf="@+id/number_layout">

        <android.support.design.widget.TextInputEditText
            android:id="@+id/edtExpiryTps"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:digits="0,1,2,3,4,5,6,7,8,9,/"
            android:hint="@string/label_mm_yy"
            android:inputType="number"
            android:maxLength="5" />
    </android.support.design.widget.TextInputLayout>


    <android.support.design.widget.TextInputLayout
        android:id="@+id/cvc_layout"
        style="@style/Beam.TextInputLayout"
        android:layout_width="0dip"
        android:layout_height="wrap_content"
        android:layout_marginBottom="@dimen/spacing_2x"
        android:layout_marginEnd="@dimen/spacing_2x"
        android:layout_marginStart="@dimen/spacing_2x"
        android:layout_marginTop="@dimen/spacing_2x"
        android:theme="@style/Beam.TextInputLayout"
        support:layout_constraintEnd_toEndOf="parent"
        support:layout_constraintStart_toEndOf="@+id/expiry_layout"
        support:layout_constraintTop_toBottomOf="@+id/number_layout">

        <android.support.design.widget.TextInputEditText
            android:id="@+id/edtCvcTps"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="@string/label_cvc"
            android:drawableEnd="@drawable/ic_payment_black_24_px"
            android:inputType="number"
            android:maxLength="3" />
    </android.support.design.widget.TextInputLayout>


    <android.support.design.widget.TextInputLayout
        android:id="@+id/full_name_layout"
        style="@style/Beam.TextInputLayout"
        android:layout_width="0dip"
        android:layout_height="wrap_content"
        android:layout_marginBottom="@dimen/spacing_2x"
        android:layout_marginEnd="@dimen/spacing_2x"
        android:layout_marginTop="@dimen/spacing_2x"
        android:theme="@style/Beam.TextInputLayout"
        support:layout_constraintEnd_toEndOf="parent"
        support:layout_constraintStart_toStartOf="@id/number_layout"
        support:layout_constraintTop_toBottomOf="@+id/cvc_layout">

        <android.support.design.widget.TextInputEditText
            android:id="@+id/edtFulltNameTps"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="@string/label_name_on_card"
            android:inputType="textCapWords"
            android:maxLines="1" />
    </android.support.design.widget.TextInputLayout>

    <ProgressBar
        android:id="@+id/viewProgress"
        android:layout_width="wrap_content"
        android:layout_height="0dip"
        android:theme="@style/Widget.AppCompat.ProgressBar"
        support:layout_constraintBottom_toBottomOf="@+id/btnSubmitTps"
        support:layout_constraintEnd_toEndOf="@+id/btnSubmitTps"
        support:layout_constraintTop_toTopOf="@+id/btnSubmitTps" />


    <Button
        android:id="@+id/btnSubmitTps"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:layout_marginBottom="@dimen/spacing_2x"
        android:layout_marginEnd="@dimen/spacing_2x"
        android:layout_marginStart="@dimen/spacing_2x"
        android:layout_marginTop="@dimen/spacing_2x"
        android:enabled="false"
        android:text="@string/label_done"
        support:layout_constraintEnd_toEndOf="parent"
        support:layout_constraintTop_toBottomOf="@id/full_name_layout" />


</android.support.constraint.ConstraintLayout>
```


### Verify Credit Card Page
In order to customize Verify Credit card ui, **tps_verify_card_layout.xml** should be added to project. TPS SDK automatically detects if tps_verify_card_layout.xml exists and render it to screen. This xml should contain this fields with specified ids.

|  View Type   | Description       |id                |
| ------------ | ----------------- |------------------ |
|  EditText | Verify Ammount    |edtVerifyAmountTps |
|  Button   | Submit Button     |edtSubmitVerifyTps |

```xml
<android.support.constraint.ConstraintLayout
  xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:support="http://schemas.android.com/apk/res-auto"
  android:layout_width="match_parent"
  android:layout_height="match_parent">

  <android.support.v7.widget.Toolbar
    android:id="@+id/viewToolbar"
    support:navigationIcon="@drawable/ic_arrow_back"
    support:layout_constraintStart_toStartOf="parent"
    support:layout_constraintEnd_toEndOf="parent"
    support:layout_constraintTop_toTopOf="parent"
    android:theme="@style/ThemeOverlay.AppCompat.Light"
    android:minHeight="?android:attr/actionBarSize"
    android:background="@color/colorPrimary"
    support:titleTextColor="@color/colorToolbarTextAndIconTint"
    android:layout_width="0dip"
    android:layout_height="wrap_content" />

  <android.support.design.widget.TextInputLayout
    android:id="@+id/viewTextInputLayout"
    style="@style/Beam.TextInputLayout"
    android:layout_marginBottom="@dimen/spacing_2x"
    android:layout_marginStart="@dimen/spacing_9x"
    android:layout_marginEnd="@dimen/spacing_2x"
    android:layout_marginTop="@dimen/spacing_2x"
    support:layout_constraintStart_toStartOf="parent"
    support:layout_constraintEnd_toEndOf="parent"
    support:layout_constraintTop_toBottomOf="@+id/viewToolbar"
    android:layout_width="0dip"
    android:layout_height="wrap_content">

    <android.support.design.widget.TextInputEditText
      android:id="@+id/viewEditText"
      android:inputType="numberDecimal"
      android:layout_width="match_parent"
      android:layout_height="wrap_content" />

  </android.support.design.widget.TextInputLayout>

  <Button
    android:id="@+id/viewButtonCharge"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_marginEnd="@dimen/spacing_2x"
    support:layout_constraintTop_toBottomOf="@+id/viewTextInputLayout"
    support:layout_constraintEnd_toEndOf="parent"
    style="@style/Beam.Button"
    android:textAllCaps="true"
    android:visibility="visible"
    android:text="@string/label_done"/>

  <ProgressBar
    android:id="@+id/viewProgress"
    android:layout_width="wrap_content"
    android:layout_height="0dip"
    android:theme="@style/Widget.AppCompat.ProgressBar"
    support:layout_constraintTop_toTopOf="@+id/viewButtonCharge"
    support:layout_constraintEnd_toEndOf="@+id/viewButtonCharge"
    support:layout_constraintBottom_toBottomOf="@+id/viewButtonCharge" />

  <TextView
    android:id="@+id/viewTextTitle"
    style="@style/Beam.Text.H4"
    android:layout_marginStart="@dimen/spacing_9x"
    android:layout_marginEnd="@dimen/spacing_2x"
    support:layout_constraintStart_toStartOf="parent"
    support:layout_constraintEnd_toEndOf="parent"
    support:layout_constraintTop_toBottomOf="@+id/viewButtonCharge"
    android:layout_width="0dip"
    android:layout_height="wrap_content"
    android:layout_marginTop="@dimen/spacing_4x"
    android:textColor="@color/beam_grey"
    android:typeface="monospace"/>

  <TextView
    style="@style/Beam.Text.S"
    android:id="@+id/viewTextMessage"
    android:layout_marginTop="@dimen/spacing_4x"
    support:layout_constraintTop_toBottomOf="@+id/viewTextTitle"
    android:layout_width="0dip"
    android:layout_height="wrap_content"
    android:layout_marginStart="@dimen/spacing_9x"
    android:layout_marginEnd="@dimen/spacing_2x"
    support:layout_constraintStart_toStartOf="parent"
    support:layout_constraintEnd_toEndOf="parent"
    android:text="@string/label_limits_apply_until_verified"
    android:textColor="@color/beam_grey_light"
    android:typeface="monospace"/>

  <TextView
    android:id="@+id/viewTextLink"
    style="@style/Beam.Text.S"
    support:layout_constraintTop_toBottomOf="@+id/viewTextMessage"
    android:layout_width="0dip"
    android:layout_height="wrap_content"
    android:layout_marginStart="@dimen/spacing_9x"
    android:layout_marginEnd="@dimen/spacing_2x"
    support:layout_constraintStart_toStartOf="parent"
    support:layout_constraintEnd_toEndOf="parent"
    android:layout_marginTop="@dimen/spacing_2x"
    android:textColor="@color/beam_grey_light"
    android:textColorHighlight="@color/beam_cta_20"
    android:textColorLink="@color/beam_cta"
    android:typeface="monospace" />

</android.support.constraint.ConstraintLayout>
```




## Version
* 1.2
