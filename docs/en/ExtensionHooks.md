# Extension hooks

You can hook into the payment process with your custom extensions.

Here's a list of all hooks available to extensions.

### Payment

 - `onAuthorized` called when a payment was successfully authorized. You'll get the `ServiceResponse` as parameter.
 - `onAwaitingAuthorized` called when authorization was completed, but is waiting for an asynchronous notification from the gateway. You'll get the `ServiceResponse` as parameter.
 - `onCaptured` called when a payment was successfully captured. You'll get the `ServiceResponse` as parameter.
 - `onAwaitingCaptured` called when a purchase completes, but is waiting for an asynchronous notification from the gateway. You'll get the `ServiceResponse` as parameter.
 - `onRefunded` called when a payment was successfully refunded. You'll get the `ServiceResponse` as parameter.
 - `onVoid` called when a payment was successfully voided/cancelled via Omnipay. You'll get the `ServiceResponse` as parameter.
 - `onCancelled` called then a payment was cancelled by the user (eg. user cancelled offsite payment). This is not an action that goes through Omnipay, so there's no parameter here.
 - `updateCMSFields` standard SilverStripe hook to update CMS fields.
 
### PaymentGatewayController

 - `updatePaymentFromRequest` called for every request that goes to the PaymentGatewayController. Can be used to return a Payment object from the request data. Needed for enabling [static routes](StaticRoutes.md)
 - `updatePaymentActionFromRequest` called for every request to the PaymentGatewayController. Can be used to set the payment action from the incoming request-data. Sometimes needed for [static routes](StaticRoutes.md)
 
### PaymentService

 - `updateServiceResponse` every service response that is being generated from the different services will be passed through this callback. You'll get the `ServiceResponse` as parameter.
 - `updatePartialPayment` whenever a partial payment is being generated "from" the main payment, this extension will be called. You'll get the new partial `Payment` as the first and the "original/parent" `Payment` as a second parameter.

##### The `updateServiceResponse` hook

Since the service-responses are crucial to a proper application-flow, you should be extremely careful when modifying a response!
The most common use-case for a modification of the service-response is probably to return a proper response to a notification coming from your payment provider.

Let's say you have two (fictional) payment providers, 'EasyPayProvider' and 'CoolPayProvider'. Both return success-status via an asynchronous notification.

The 'EasyPayProvider' API states, that you should return a HTTP Response with Status `200`, and a message body of `SUCCESS` when payment notification was handled successfully.

The 'CoolPayProvider' API states, that you should return an HTTP Response with Status `200`, a message body of `OK` and a response header: `X-CoolPay-Success: 1`

Here's how you can achieve that with an Extension:

```php
use SilverStripe\Omnipay\Service\ServiceResponse;

class NotifyResponseExtension extends Extension
{
    public function updateServiceResponse(ServiceResponse $serviceResponse)
    {
        if ($serviceResponse->isNotification()) {
            if ($serviceResponse->getPayment()->Gateway == 'EasyPayProvider') {
                $serviceResponse->setHttpResponse(new SS_HTTPResponse('SUCCESS', 200));
            } else if ($serviceResponse->getPayment()->Gateway == 'CoolPayProvider') {
                $httpResponse = new SS_HTTPResponse('OK', 200);
                $httpResponse->addHeader('X-CoolPay-Success', '1');
                $serviceResponse->setHttpResponse($httpResponse);
            }
        }
    }
}
```

And then add the extension as usual:
```yml
SilverStripe\Omnipay\Service\PaymentService:
  extensions:
    - NotifyResponseExtension
```

##### The `updatePartialPayment` hook

Partial payments (eg. partial refunds and partial captures) generate new Payment instances.
These new payments will inherit the `Gateway`, `TransactionReference`, `SuccessUrl` and `FailureUrl` of the initial payment.

You'll get the newly created Payment instance as parameter in the `updatePartialPayment` hook (right before it will be written to DB).
The original Payment (eg. the "parent" payment) is passed as the second parameter to the extension hook.

If you added custom properties to your Payment (eg. via Extension) it is your duty as developer to copy these to the newly created Payment, if necessary! 

*Please do not tinker with the default fields of the Payment class, as further processing of the Payments depend on these values to be correct!*

### AuthorizeService

 - `onBeforeAuthorize` called just before the `authorize` call is being made to the gateway. Passes the Gateway-Data (an array) as parameter, which allows you to modify the gateway data prior to being sent.
 - `onAfterAuthorize` called just after the Omnipay `authorize` call. Will pass the Omnipay request object as parameter.
 - `onAfterSendAuthorize` called after `send` has been called on the Omnipay request object. You'll get the request as first, and the omnipay response as second parameter.
 - `onBeforeCompleteAuthorize` called just before the `completeAuthorize` call is being made to the gateway. Passes the Gateway-Data (an array) as parameter, which allows you to modify the gateway data prior to being sent.
 - `onAfterCompleteAuthorize` called just after the Omnipay `completeAuthorize` call. Will pass the Omnipay request object as parameter.

### CaptureService

 - `onBeforeCapture` called just before the `capture` call is being made to the gateway. Passes the Gateway-Data (an array) as parameter, which allows you to modify the gateway data prior to being sent.
 - `onAfterCapture` called just after the Omnipay `capture` call. Will pass the Omnipay request object as parameter.
 - `onAfterSendCapture` called after `send` has been called on the Omnipay request object. You'll get the request as first, and the omnipay response as second parameter.

### PurchaseService

 - `onBeforePurchase` called just before the `purchase` call is being made to the gateway. Passes the Gateway-Data (an array) as parameter, which allows you to modify the gateway data prior to being sent.
 - `onAfterPurchase` called just after the Omnipay `purchase` call. Will pass the Omnipay request object as parameter.
 - `onAfterSendPurchase` called after `send` has been called on the Omnipay request object. You'll get the request as first, and the omnipay response as second parameter.
 - `onBeforeCompletePurchase` called just before the `completePurchase` call is being made to the gateway. Passes the Gateway-Data (an array) as parameter, which allows you to modify the gateway data prior to being sent.
 - `onAfterCompletePurchase` called just after the Omnipay `completePurchase` call. Will pass the Omnipay request object as parameter.

### CaptureService

 - `onBeforeRefund` called just before the `refund` call is being made to the gateway. Passes the Gateway-Data (an array) as parameter, which allows you to modify the gateway data prior to being sent.
 - `onAfterRefund` called just after the Omnipay `refund` call. Will pass the Omnipay request object as parameter.
 - `onAfterSendRefund` called after `send` has been called on the Omnipay request object. You'll get the request as first, and the omnipay response as second parameter.

### VoidService

 - `onBeforeVoid` called just before the `void` call is being made to the gateway. Passes the Gateway-Data (an array) as parameter, which allows you to modify the gateway data prior to being sent.
 - `onAfterVoid` called just after the Omnipay `void` call. Will pass the Omnipay request object as parameter.
 - `onAfterSendVoid` called after `send` has been called on the Omnipay request object. You'll get the request as first, and the omnipay response as second parameter.

### ServiceFactory

You can use extension hooks to override what services are being created for what intent.

If somebody does the following:

```php
$factory = ServiceFactory::create();
$service = $factory->getService($payment, ServiceFactory::INTENT_PAYMENT);
```

The constant `ServiceFactory::INTENT_PAYMENT` just translates to a string `"payment"`, which invokes the following
hook on any ServiceFactory extension: `createPaymentService`. The hook will get the `Payment` object as parameter and
should return a `PaymentService` instance (eg. a subclass).

Example code that might be in your extension:

```php
// This is just an example and already implemented in the default Factory, do not create an actual extension to do this.
public function createPaymentService(Payment $payment)
{
    if (GatewayInfo::shouldUseAuthorize($payment->Gateway)) {
        return AuthorizeService::create($payment);
    } else {
        return PurchaseService::create($payment);
    }
 }
```

Please be aware that you can't implement the same create-method in multiple extensions. If the factory encounters
several Extensions with the same method, it will raise an exception.
