# 如何给你的网站增加推送通知

这周我发布了 [https://github-notifications.herokuapp.com/](https://github-notifications.herokuapp.com/)，一个 [Firefox 推送通知](http://jbalogh.me/2012/01/30/push-notifications/)的实例。 它使用了 Github 的网站接口来实现当你的代码库有新的提交时发送通知。推送通知在 Firefox 中已经以一个试验性[插件](https://github.com/jbalogh/push-addon/)实现了。

这篇文章将展示我用于从网站上发送推送通知的代码。

## 获取推送 URL

当你给予一个网站权限来发送推送通知时，Firefox 向推送通知服务 (Push Notification Service) 请求创建一个 URL 以使网站可以联系到你。这个 URL 用过 `mozNotification` JavaScript API 返回。

	var notification = navigator.mozNotification;
	if (notification && notification.requestRemotePermission) {
	  var request = notification.requestRemotePermission();
	  request.onsuccess = function() {
	    var url = request.result.url;
	    jQuery.post("/add-push-url", {"push-url": url});
	  }
	}

这段代码检查了 `navigator.mozNotification` 的存在，并且使用 `requestRemotePermission()` 请求发送通知的权限。当 `onsuccess` 事件被触发时，回调函数将这个推送用的 URL `POST` 回服务器。

(你可以通过安装这个[插件](https://github.com/jbalogh/push-addon/)来实验 `mozNotification` API。)

## 保存推送 URL

我的网站的后端是一个简单的 [Flask](http://flask.pocoo.org/) 应用。`User` 模型保存了用户名和推送 URL：

	class User(Model, db.Model):
	    id = db.Column(db.Integer, primary_key=True)
	    username = db.Column(db.String(80), unique=True)
	    push_url = db.Column(db.String(256), nullable=True)

当通过 OAuth 连接到 Github 时，将会创建一个 `push_url` 为空的新用户，而 `push_url` 将在调用 `/add-push-url` 视图时被填充：

	@app.route('/add-push-url', methods=['POST'])
	def add_push_url():
	    username = session['username']
	    user = User.query.filter_by(username=username).first_or_404()
	    user.push_url = request.form['push-url']
	    db.session.add(user)
	    db.session.commit()
	    notify(user.push_url,
	           'Welcome to Github Notifications!',
	           'So glad to have you %s.' % user.username)
	    return ''

## 发送通知

发送一条通知非常简单，只要 `POST` 到推送 URL 即可。

	def notify(push_url, title, body, action_url=None):
	    msg = {'title': title,
	           'body': body}
	    if action:
	        msg['actionUrl'] = action_url
	    requests.post(push_url, msg)

`notify` 函数使用 [requests](http://python-requests.org/) 库将一条消息以 URL 参数编码的字符串发送回用户的推送 URL，在那之后，推送通知系统负责将消息显示在用户的浏览器上。

以上就是发送推送通知到 Firefox 需要的全部代码，我们努力让整个系统对于开发者来说尽量简单。在未来的数个月中，这个插件将会被集成到浏览器中，而我们的推送通知服务也将上线。如果你有任何问题，欢迎通过电子邮件或 Twitter 与我联系。