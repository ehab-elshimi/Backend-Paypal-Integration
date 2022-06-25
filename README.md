# Laravel Paypal Integration
Simple paypal integration with laravel framework
![LaravelPaypal](https://abdelrahmanetry.info/assets/github/paypal_laravel.png)

## Requirements
- Create a business account on paypal from this link https://developer.paypal.com
- Activate your Sandbox account to get access to some important details and passwords that you will need in the process of integrating PayPal Payment Gateway in Laravel.
- Locate Both `client_id` and `secret_key` in your profile.

## Usage
You can clone this repository for a simple paypal integration sample or you can do the following:-

1. Create new laravel project by running the following command:
    ```bash
    composer create-project laravel/laravel paypal-integration
    ```
2. Go to your project's directory 
    ```bash
    cd paypal-integration
    ```
3. Install PayPal’s SDK in your project 
    ```bash
    composer require srmklive/PayPal
    ```
4. After installation progress is completed, register your paypal account to your application by opening the project in code editor and go to `config/app.php`.

5. Add the following to your service provider array 
    ~~~
    'providers' => [
        ...
        Srmklive\PayPal\Providers\PayPalServiceProvider::class
    ]
    ~~~

6. Then add the following code in the aliases array in the same page
    ~~~
    'aliases' => [
        ...
        'PayPal' => Srmklive\PayPal\Facades\PayPal::class
    ]
    ~~~

7. Now in order to integrate with paypal you'll need to add some credentials in your `.env` file. You can get the credentials from your paypal account.
    ```
    PAYPAL_MODE=sandbox
    PAYPAL_SANDBOX_API_USERNAME=xxxxx-xxx-xx
    PAYPAL_SANDBOX_API_PASSWORD=xxxxxxxxxx
    PAYPAL_SANDBOX_API_SECRET=xxxxxxx-xxxxx-xxx
    PAYPAL_CURRENCY=USD
    PAYPAL_SANDBOX_API_CERTIFICATE=
    
    PAYPAL_SANDBOX_CLIENT_ID=Aa3Yg1lziML0Fg1aMOW_nEY-C7zRbHnEJ5XxbmSJ

    PAYPAL_SANDBOX_CLIENT_SECRET=EOWVsp68LwKE4tuAMS0IkzeJbo5zcplEY5ZuY4_Z
    ```
8. Now run the following command to create the controller:
    ```bash
    php artisan make:controller PaypalController
    ```
    then place the following code
    ```php
        <?php
        namespace App\Http\Controllers;
        use Illuminate\Http\Request;
        use Srmklive\PayPal\Services\PayPal as PayPalClient;
    
        class PayPalController extends Controller{
            public function createTransaction(){
                return view('transaction');
            }
            
            /**
            * process transaction.
            *
            * @return \Illuminate\Http\Response
            */
            
            public function processTransaction(Request $request){
                $provider = new PayPalClient;
                $provider->setApiCredentials(config('paypal'));
                $paypalToken = $provider->getAccessToken();
        
                $response = $provider->createOrder([
                    "intent" => "CAPTURE",
                    "application_context" => [
                        "return_url" => route('successTransaction'),
                        "cancel_url" => route('cancelTransaction'),
                    ],
                    "purchase_units" => [
                        0 => [
                            "amount" => [
                                "currency_code" => "USD",
                                "value" => "1000.00"
                            ]
                        ]
                    ]
                ]);
    
                if (isset($response['id']) && $response['id'] != null) {
                    // redirect to approve href
                    foreach ($response['links'] as $links) {
                        if ($links['rel'] == 'approve') {
                            return redirect()->away($links['href']);
                        }
                    }
        
                    return redirect()
                        ->route('createTransaction')
                        ->with('error', 'Something went wrong.');
                } else {
                    return redirect()
                        ->route('createTransaction')
                        ->with('error', $response['message'] ?? 'Something went wrong.');
                }
            }
    
            /**
             * success transaction.
             *
             * @return \Illuminate\Http\Response
             */
            public function successTransaction(Request $request){
                $provider = new PayPalClient;
                $provider->setApiCredentials(config('paypal'));
                $provider->getAccessToken();
                $response = $provider->capturePaymentOrder($request['token']);
        
                if (isset($response['status']) && $response['status'] == 'COMPLETED') {
                    return redirect()
                        ->route('createTransaction')
                        ->with('success', 'Transaction complete.');
                } else {
                    return redirect()
                        ->route('createTransaction')
                        ->with('error', $response['message'] ?? 'Something went wrong.');
                }
            }
        
            /**
             * cancel transaction.
             *
             * @return \Illuminate\Http\Response
             */
            public function cancelTransaction(Request $request){
                return redirect()
                    ->route('createTransaction')
                    ->with('error', $response['message'] ?? 'You have canceled the transaction.');
            }
        }
    ```

9. Then you'll need to create a blade view by creating a file named `transaction.blade.php` inside **resources/views folder**
    ```html
     <!doctype html>
    <html>
        <head>
            <meta charset="utf-8">
            <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
            <title>Pay $1000</title>
            <script src="https://www.paypal.com/sdk/js?client-id={{ env('PAYPAL_SANDBOX_CLIENT_ID') }}"></script>
        </head>

        <body>
            <a class="btn btn-primary m-3" href="{{ route('processTransaction') }}">Pay $1000</a>
            @if(\Session::has('error'))
            <div class="alert alert-danger">{{ \Session::get('error') }}</div>
            {{ \Session::forget('error') }}
            @endif
    
            @if(\Session::has('success'))
            <div class="alert alert-success">{{ \Session::get('success') }}</div>
            {{ \Session::forget('success') }}
            @endif
        </body>
    </html>
    ```
10. Finally, create the routes by going to **routes/web.php** and paste the following
    ```php 
    Route::get('/', function () {
        return view('transaction');
    });
    Route::get('create-transaction'     ,   'PayPalController@createTransaction')->name('createTransaction');
    Route::get('process-transaction'    ,   'PayPalController@processTransaction')->name('processTransaction');
    Route::get('success-transaction'    ,   'PayPalController@successTransaction')->name('successTransaction');
    Route::get('cancel-transaction'     ,   'PayPalController@cancelTransaction')->name('cancelTransaction');
    ```

11. You can test your code by running the **artisan serve command** 
    ```bash 
    php artisan serve 
    ```

