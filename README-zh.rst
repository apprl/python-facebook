Python Facebook

使用 `Python` 封装的 `Facebook` 平台下的一些数据接口

.. image:: https://travis-ci.org/sns-sdks/python-facebook.svg?branch=master
    :target: https://travis-ci.org/sns-sdks/python-facebook
    :alt: Build Status

.. image:: https://readthedocs.org/projects/python-facebook-api/badge/?version=latest
    :target: https://python-facebook-api.readthedocs.io/en/latest/?badge=latest
    :alt: Documentation Status

.. image:: https://codecov.io/gh/sns-sdks/python-facebook/branch/master/graph/badge.svg
    :target: https://codecov.io/gh/sns-sdks/python-facebook
    :alt: Codecov

.. image:: https://img.shields.io/pypi/v/python-facebook-api.svg
    :target: https://pypi.org/project/python-facebook-api
    :alt: PyPI

README: `English <README.rst>`_ | `中文 <README-zh.rst>`_

====
致谢
====

灵感来自 `Python-Twitter <https://github.com/bear/python-twitter>`_.

====
说明
====

该库提供一种更加简单的方式去使用 ``Facebook`` 平台的数据接口。 注意，当你使用时，由于一些原因，需要在外网环境下才可以使用。

目前包括了 ``Facebook``, ``Instagram Business`` 产品数据的使用。

====
安装
====

现在可以使用 `pip` 来安装这个包啦::

    $pip install --upgrade python-facebook-api
    ✨🍰✨

====
文档
====

你可以访问: https://python-facebook-api.readthedocs.io/en/latest/ 去查看相关文档 (Doing).

涉及的 ``Facebook Graph API`` 你都可以通过访问: https://developers.facebook.com/docs/graph-api/ 去查看
涉及的 ``Instagram Graph API`` 你都可以通过访问: https://developers.facebook.com/docs/instagram-api/ 去查看

=======================
使用 Facebook Graph API
=======================

API 通过 ``pyfacebook.Api`` 类公开访问。

为了获取对应的数据，你首先需要一个 Facebook App。
你可以访问 `App docs <https://developers.facebook.com/docs/apps>`_ 去获取更多关于如何创建 App 和 如何为 App 申请相关的权限的信息。

----------
初始化 API
----------

Facebook 存在不同类型的访问口令。使用不同的访问口令可以获取不同类型的数据。

1. 用户访问口令
#. 应用访问口令
#. 主页访问口令
#. 客户端访问口令 (由于用到该口令的地方很少，此库不提供)

你可以阅读有关 `访问口令`_ 的文档去获取更多的信息。

如果你想要通过用户进行授权来获取用户的访问口令，你可以按照 `手动授权`_ 的文档来初始化 Api。

如果你只是想通过应用访问口令来获取一些公开数据，你可以通过如下方式初始化 Api::

    In [2]: api = Api(app_id="your app id", app_secret="your app secret", application_only_auth=True)
    In [3]: api.get_token_info()  # 获取当前口令的信息
    Out[3]: AccessToken(app_id='id', application='app name', user_id=None)

如果你已经有了一个短期的访问口令，你可以通过如下方式初始化 Api::

    In [4]: api = Api(app_id="your app id", app_secret="your app secret", short_token="short-lived token")
    In [5]: api.get_token_info()
    Out[5]: AccessToken(app_id='id', application='app name', user_id='token user id')

如果你已经有了一个长期的访问口令，你可以通过如下方式初始化 Api
(注意，只提供一个 ``long_term_token``参数已经足以初始化，当时为了安全器件，最好还是提供一下 app 的认证数据)::

    In [6]: api = Api(app_id="your app id", app_secret="your app secret", long_term_token="long-term token")
    In [7]: api.get_token_info()
    Out[7]: AccessToken(app_id='id', application='app name', user_id='token user id')

    #  只使用 ``long_term_token`` 参数时，需要该口令具有 ``manage_pages`` 的权限
    In [8]: api = Api(long_term_token="long-term token")


使用短期口令和长期口令初始化的区别在于，使用短期口令时，库会自动获取到 长期的口令。

--------
获取数据
--------

你可以通过如下的方式来获取一个主页的公开数据。

获取单个主页的公开数据::

    In [3]: api.get_page_info(username='facebookapp')
    Out[3]: Page(id='20531316728', name='Facebook', username='facebookapp')

仅通过一次请求来获取多个主页的公开数据(参数 ``ids`` 可以是主页 ID 和主页用户名混用的列表)::

    In [4]: api.get_pages_info(ids=["20531316728", "nba"])
    Out[4]:
    {'20531316728': Page(id='20531316728', name='Facebook', username='facebookapp'),
     'nba': Page(id='8245623462', name='NBA', username='nba')}

存在多种方法来获取主页的贴文数据。

>>> api.get_page_feeds()
>>> api.get_page_posts()
>>> api.get_page_published_posts()
>>> api.get_page_tagged_posts()

主页 feeds 可以获取主页或者由此主页上的其他人发布的帖子动态(包括状态更新)和链接::

    In [5]: api.get_page_feeds(page_id="20531316728",count=2)
    Out[5]:
    [Post(id='20531316728_587455038708591', permalink_url='https://www.facebook.com/facebookapp/videos/587455038708591/'),
     Post(id='20531316728_10159023836696729', permalink_url='https://www.facebook.com/20531316728/posts/10159023836696729/')]

主页 posts 只能获取到由该主页发布的帖子::

    In [6]: api.get_page_posts(page_id="20531316728",count=2)
    Out[6]:
    [Post(id='20531316728_587455038708591', permalink_url='https://www.facebook.com/facebookapp/videos/587455038708591/'),
     Post(id='20531316728_10159023836696729', permalink_url='https://www.facebook.com/20531316728/posts/10159023836696729/')]

因为图谱 API 的限制. `动态 <https://developers.facebook.com/docs/graph-api/reference/v5.0/page/feed>`_。
API 每年返回大约 600 个经排名的帖子。

所以如果你想要获取主页的所有帖子或者标记该主页的帖子。你需要使用 ``get_page_published_posts`` 方法，该方法需要你的访问口令带有 ``manage_pages`` 的权限。

你可以通过授权来得到这样的访问口令，按照 `手动授权`_ 的文档即可。

之后你可以获取到主页所有帖子::

    In [7]: api.get_published_posts(username='facebookapp', access_token='page access token')
    Out[7]: [Post...]

获取标记该主页的帖子::

    In [8]: api.get_tagged_posts(username='facebookapp', access_token='page access token')
    Out[8]: [Post...]


如果你已经有了贴文的 ID，你可以通过如下方法来获取贴文的详情数据。

获取单个贴子的数据::

    In [9]: api.get_post_info(post_id="20531316728_587455038708591")
    Out[9]: Post(id='20531316728_587455038708591', permalink_url='https://www.facebook.com/facebookapp/videos/587455038708591/')

单请求获取多个贴子的数据::

    In [10]: api.get_posts_info(ids=["20531316728_587455038708591", "20531316728_10159023836696729"])
    Out[10]:
    {'20531316728_587455038708591': Post(id='20531316728_587455038708591', permalink_url='https://www.facebook.com/facebookapp/videos/587455038708591/'),
     '20531316728_10159023836696729': Post(id='20531316728_10159023836696729', permalink_url='https://www.facebook.com/20531316728/posts/10159023836696729/')}


你可以通过资源对象(帖子，图片等)的 ID 来获取对应的评论数据::

    In [11]: api.get_comments_by_object(object_id="20531316728_587455038708591", count=2)
    Out[11]:
    ([Comment(id='587455038708591_587460942041334', can_like=True, can_comment=True, comment_count=2, like_count=1),
      Comment(id='587455038708591_587464298707665', can_like=True, can_comment=True, comment_count=2, like_count=14)],
     CommentSummary(total_count=392, can_comment=True))

如果你已经有了评论的 ID，你可以通过如下方式来获取评论的详情数据::

获取单个评论的数据::

    In [12]: api.get_comment_info(comment_id="587455038708591_587460942041334")
    Out[12]: Comment(id='587455038708591_587460942041334', comment_count=2, like_count=1)

单请求获取多个评论的数据::

    In [13]: api.get_comments_info(ids=["587455038708591_587460942041334", "587455038708591_587464298707665"])
    Out[13]:
    {'587455038708591_587460942041334': Comment(id='587455038708591_587460942041334', comment_count=2, like_count=1),
     '587455038708591_587464298707665': Comment(id='587455038708591_587464298707665', comment_count=2, like_count=14)}


你可以通过如下方式来获取主页的头像。

获取单个主页的头像数据::

    In [14]: api.get_picture(page_id="20531316728")
    Out[14]: ProfilePictureSource(url='https://scontent.xx.fbcdn.net/v/t1.0-1/p100x100/58978526_10158354585751729_7411073224387067904_o.png?_nc_cat=1&_nc_oc=AQmaFO7eND-DVRoArrQLUZVDpmemw8nMPmHJWvoCyXId_MKLLHQdsS8UbTOX4oaEfeQ&_nc_ht=scontent.xx&oh=128f57c4dc65608993af62b562d92d84&oe=5E942420', height=100, width=100)


单请求获取多个主页的头像数据::

    In [15]: api.get_pictures(ids=["20531316728", "nba"])
    Out[15]:
    {'20531316728': ProfilePictureSource(url='https://scontent.xx.fbcdn.net/v/t1.0-1/p100x100/58978526_10158354585751729_7411073224387067904_o.png?_nc_cat=1&_nc_oc=AQmaFO7eND-DVRoArrQLUZVDpmemw8nMPmHJWvoCyXId_MKLLHQdsS8UbTOX4oaEfeQ&_nc_ht=scontent.xx&oh=128f57c4dc65608993af62b562d92d84&oe=5E942420', height=100, width=100),
     'nba': ProfilePictureSource(url='https://scontent.xx.fbcdn.net/v/t1.0-1/p100x100/81204460_10158199356848463_5727214464013434880_n.jpg?_nc_cat=1&_nc_oc=AQmcent57E-a-923C_VVpiX26nGqKDodImY1gsiu7h1czDmcpLHXR8D5hIh9g9Ao3wY&_nc_ht=scontent.xx&oh=1656771e6c11bd03147b69ee643238ba&oe=5E66450C', height=100, width=100)}

========================
使用 Instagram Graph API
========================

Instagram 图谱 API 可以 `instagram Professional accounts <https://help.instagram.com/502981923235522>`_ (商家和创作者) 的数据。

----------
初始化 Api
----------

和 Facebook 的图谱 API 的类似，你可以通过多种方式来初始化 Api。但是你只能使用用户访问口令，并且需要你的商务帐号 ID。

如果你想要通过授权来获取用户的访问口令，你可以按照 `手动授权`_ 来初始化 api。

如果你已经有了一个短期的访问口令，你可以通过如下方式初始化 Api::

    In [2]: api = IgProApi(app_id="your app id", app_secret="your app secret", short_token="short-lived token", instagram_business_id="17841406338772941")
    In [3]: api.get_token_info()
    Out[3]: AccessToken(app_id='id', application='app name', user_id="token user id")

如果你已经有了一个长期的访问口令，你可以通过如下方式初始化 Api
(注意，只提供一个 ``long_term_token``参数已经足以初始化，当时为了安全器件，最好还是提供一下 app 的认证数据)::

    In [4]: api = IgProApi(app_id="your app id", app_secret="your app secret", long_term_token="long-lived token")
    In [5]: api.get_token_info()
    Out[5]: AccessToken(app_id='id', application='app name', user_id='token user id')

--------
获取数据
--------

如果你想要搜索其他商家帐号的基础数据和帖子。你可以使用如下的方法::

    - discovery_user: 获取用户的基础数据
    - discovery_user_medias: 获取用户的贴文

.. note::
   使用 discovery 方法进行搜索只支持使用用户名

通过其他商家用户的用户名来获取基础数据::

    In [6]: api.discovery_user(username="facebook")
    Out[6]: IgProUser(id='17841400455970028', name='Facebook', username='facebook')

通过其他商家用户的用户名来获取贴文数据::

    In [7]: api.discovery_user_medias(username="facebook", count=2)
    Out[7]:
    [IgProMedia(comments=None, id='17859633232647524', permalink='https://www.instagram.com/p/B6jje2UnoH8/'),
     IgProMedia(comments=None, id='18076151185161297', permalink='https://www.instagram.com/p/B6ji-PZH2V1/')]

获取你的帐号的信息::

    In [10]: api.get_user_info(user_id="your instagram business id")
    Out[10]: IgProUser(id='17841406338772941', name='LiuKun', username='ikroskun')

获取你的贴文::

    In [11]: api.get_user_medias(user_id=api.instagram_business_id, count=2)
    Out[11]:
    [IgProMedia(comments=None, id='18075344632131157', permalink='https://www.instagram.com/p/B38X8BzHsDi/'),
     IgProMedia(comments=None, id='18027939643230671', permalink='https://www.instagram.com/p/B38Xyp6nqsS/')]

如果你已经有了一些贴文的 ID 你可以通过如下方式获取贴文的详情数据。

获取单个贴文的详情信息::

    In [12]: api.get_media_info(media_id="18075344632131157")
    Out[12]: IgProMedia(comments=None, id='18075344632131157', permalink='https://www.instagram.com/p/B38X8BzHsDi/')


通过单个请求获取多个贴文的详情数据::

    In [13]: api.get_medias_info(media_ids=["18075344632131157", "18027939643230671"])
    Out[13]:
    {'18075344632131157': IgProMedia(comments=None, id='18075344632131157', permalink='https://www.instagram.com/p/B38X8BzHsDi/'),
     '18027939643230671': IgProMedia(comments=None, id='18027939643230671', permalink='https://www.instagram.com/p/B38Xyp6nqsS/')}


获取某个贴文的评论数据::

    In [16]: api.get_comments_by_media(media_id="17955956875141196", count=2)
    Out[16]:
    [IgProComment(id='17862949873623188', timestamp='2020-01-05T05:58:47+0000'),
     IgProComment(id='17844360649889631', timestamp='2020-01-05T05:58:42+0000')]


如果你已经有了一些评论的的 ID，你可以通过如下方式来获取评论详情。

获取单个评论的详情::

    In [17]: api.get_comment_info(comment_id="17862949873623188")
    Out[17]: IgProComment(id='17862949873623188', timestamp='2020-01-05T05:58:47+0000')

通过单个请求获取多个评论的详情::

    In [18]: api.get_comments_info(comment_ids=["17862949873623188", "17844360649889631"
    ...: ])
    Out[18]:
    {'17862949873623188': IgProComment(id='17862949873623188', timestamp='2020-01-05T05:58:47+0000'),
     '17844360649889631': IgProComment(id='17844360649889631', timestamp='2020-01-05T05:58:42+0000')}

获取某个评论的回复::

    In [19]: api.get_replies_by_comment("17984127178281340", count=2)
    Out[19]:
    [IgProReply(id='18107567341036926', timestamp='2019-10-15T07:06:09+0000'),
     IgProReply(id='17846106427692294', timestamp='2019-10-15T07:05:17+0000')]

如果你已经有了一些评论的 ID，你可以通过如下方法来获取回复详情。

获取单个评论的详情::

    In [20]: api.get_reply_info(reply_id="18107567341036926")
    Out[20]: IgProReply(id='18107567341036926', timestamp='2019-10-15T07:06:09+0000')

通过单个请求获取多个回复的详情::

    In [21]: api.get_replies_info(reply_ids=["18107567341036926", "17846106427692294"])
    Out[21]:
    {'18107567341036926': IgProReply(id='18107567341036926', timestamp='2019-10-15T07:06:09+0000'),
     '17846106427692294': IgProReply(id='17846106427692294', timestamp='2019-10-15T07:05:17+0000')}

获取授权用户的 insights 数据::

    In [4]: api.get_user_insights(user_id=api.instagram_business_id, period="day", metrics=["impressions", "reach"])
    Out[4]:
    [IgProInsight(name='impressions', period='day', values=[IgProInsightValue(value=1038, end_time='2020-01-08T08:00:00+0000'), IgProInsightValue(value=136, end_time='2020-01-09T08:00:00+0000')]),
     IgProInsight(name='reach', period='day', values=[IgProInsightValue(value=751, end_time='2020-01-08T08:00:00+0000'), IgProInsightValue(value=54, end_time='2020-01-09T08:00:00+0000')])]


获取授权用户的贴文的 insights 数据::

    In [6]: api.get_media_insights(media_id="media_id", metrics=["engagement","impressions"])
    Out[6]:
    [IgProInsight(name='engagement', period='lifetime', values=[IgProInsightValue(value=90, end_time=None)]),
     IgProInsight(name='impressions', period='lifetime', values=[IgProInsightValue(value=997, end_time=None)])]

====
TODO
====

---------
当前的功能
---------

Facebook：

- 主页信息
- 主页图片信息
- 贴文数据
- 评论数据

Instagram：

- 搜索其他业务主页的基础信息和贴文
- 获取授权业务主页的基础信息
- 获取授权业务主页的贴文信息
- 获取授权业务主页的贴文评论数据
- 获取授权业务主页的评论的回复数据
- 获取授权业务主页用户的 Insights 数据
- 获取授权业务主页贴文的 Insights 数据

----
待做
----

- Insights 数据的获取
- 发布帖子


.. _访问口令: https://developers.facebook.com/docs/facebook-login/access-tokens
.. _手动授权: https://developers.facebook.com/docs/facebook-login/manually-build-a-login-flow