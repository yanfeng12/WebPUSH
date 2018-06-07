# WebPUSH
Your First Web Push Notification
## 向网络应用添加推送通知
1. 注册服务工作线程
scripts/main.js：
'''
if ('serviceWorker' in navigator && 'PushManager' in window) {
  console.log('Service Worker and Push is supported');

  navigator.serviceWorker.register('sw.js')
  .then(function(swReg) {
    console.log('Service Worker is registered', swReg);

    swRegistration = swReg;
  })
  .catch(function(error) {
    console.error('Service Worker Error', error);
  });
} else {
  console.warn('Push messaging is not supported');
  pushButton.textContent = 'Push Not Supported';
}
'''
2. 获取应用服务器密钥
您可以在这里生成一个公私密钥对:https://web-push-codelab.glitch.me/
将公钥复制到 scripts/main.js 替换 <Your Public Key> 值：
3. 检查用户是否订阅推送，并更新按钮文字
我们将在 scripts/main.js 中创建两个函数，一个称为 initialiseUI，会检查用户当前有没有订阅，另一个称为 updateBtn，将启用我们的按钮，以及更改用户是否订阅的文本。
'''
    function initialiseUI() {
  // Set the initial subscription value
  swRegistration.pushManager.getSubscription()
  .then(function(subscription) {
    isSubscribed = !(subscription === null);

    if (isSubscribed) {
      console.log('User IS subscribed.');
    } else {
      console.log('User is NOT subscribed.');
    }

    updateBtn();
  });
}
'''
'''
    function updateBtn() {
  if (isSubscribed) {
    pushButton.textContent = 'Disable Push Messaging';
  } else {
    pushButton.textContent = 'Enable Push Messaging';
  }

  pushButton.disabled = false;
}
'''
4. 订阅用户
向 initialiseUI() 函数中的按钮添加点击侦听器
'''
function initialiseUI() {
  pushButton.addEventListener('click', function() {
    pushButton.disabled = true;
    if (isSubscribed) {
      // TODO: Unsubscribe user
    } else {
      subscribeUser();
    }
  });

  // Set the initial subscription value
  swRegistration.pushManager.getSubscription()
  .then(function(subscription) {
    isSubscribed = !(subscription === null);

    updateSubscriptionOnServer(subscription);

    if (isSubscribed) {
      console.log('User IS subscribed.');
    } else {
      console.log('User is NOT subscribed.');
    }

    updateBtn();
  });
}

'''
我们会在知道用户当前没有订阅时调用 subscribeUser()
'''
function subscribeUser() {
  const applicationServerKey = urlB64ToUint8Array(applicationServerPublicKey);
  swRegistration.pushManager.subscribe({
    userVisibleOnly: true,
    applicationServerKey: applicationServerKey
  })
  .then(function(subscription) {
    console.log('User is subscribed:', subscription);

    updateSubscriptionOnServer(subscription);

    isSubscribed = true;

    updateBtn();
  })
  .catch(function(err) {
    console.log('Failed to subscribe the user: ', err);
    updateBtn();
  });
}
'''
5. 处理推送事件
将以下代码添加到 sw.js 文件
'''
self.addEventListener('push', function(event) {
  console.log('[Service Worker] Push Received.');
  console.log(`[Service Worker] Push had this data: "${event.data.text()}"`);

  const title = 'Push Codelab';
  const options = {
    body: 'Yay it works.',
    icon: 'images/icon.png',
    badge: 'images/badge.png'
  };

  event.waitUntil(self.registration.showNotification(title, options));
});
'''
6. 通知点击
在 sw.js 中添加 notificationclick 侦听器
'''
self.addEventListener('notificationclick', function(event) {
  console.log('[Service Worker] Notification click Received.');

  event.notification.close();

  event.waitUntil(
    clients.openWindow('https://developers.google.com/web/')
  );
});
'''
6. 发送推送消息
复制页面底部的
'''
{"endpoint":"https://fcm.googleapis.com/fcm/send/fa6wLtztnJI:APA91bFdsrppaY9dM0qJ_Nwq5WgOijchysHyKAiEz2BPE3xZT3-aUZ632FaGHGMlcm1IyWiIPXLp9K_lqAfIRp9mOr29xlsXQ46LOMYDOFJ0Qaxm3Rc9l66cTbMnOcOR9hox33UP-xvf","expirationTime":null,"keys":{"p256dh":"BF5XBTHQKP-UCf9zrbp7Cbf_I6KlbWJrxS8zVdOuL1AnyWHHtnsTx9cpNnvfOACE8bZt_zd0b5Y4QCzVAVU4X8c","auth":"SXC5iC_ir157qmlSjaLhnQ"}}
粘贴到配套网站的 Subscription to Send To 文本区域
在 Text to Send 下添加您想要与推送消息一起发送的任意字符串，最后点击 Send Push Message 按钮.

'''