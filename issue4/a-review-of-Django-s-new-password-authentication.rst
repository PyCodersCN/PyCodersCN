Django 新的密码验证方法评测
===========================

Django刚刚发布的 1.4 版本中包含了一些安全方面的重要提升。其中一个是使用 PBKDF2 密码加密算法代替了 SHA1 。另外一个特性是你可以添加自己的密码加密方法。

Django会使用你提供的第一个密码加密方法（在你的 ``setting.py`` 文件里要至少有一个方法）

::
    
    PASSWORD_HASHERS = (
    'django.contrib.auth.hashers.PBKDF2PasswordHasher',
    'django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher',
    'django.contrib.auth.hashers.BCryptPasswordHasher',
    'django.contrib.auth.hashers.SHA1PasswordHasher', # Insecure Hashes
    'django.contrib.auth.hashers.MD5PasswordHasher', # Insecure Hashes
    'django.contrib.auth.hashers.CryptPasswordHasher', # Insecure Hashes
    )

我们先从一睹如何创建自己的密码加密方法开始。

首先，你必须从 ``BasePasswordHarsher`` 这里继承，它就在 ``django.contrib.auth.hashers`` 里。这个  Class 看起来就像下面这样：

::
    
    class BasePasswordHasher(object):
        """
        Abstract base class for password hashers
    
        When creating your own hasher, you need to override algorithm,
        verify(), encode() and safe_summary().
    
        PasswordHasher objects are immutable.
        """
        algorithm = None
        library = None
    
        def _load_library(self):
            if self.library is not None:
                if isinstance(self.library, (tuple, list)):
                    name, mod_path = self.library
                else:
                    name = mod_path = self.library
                try:
                    module = importlib.import_module(mod_path)
                except ImportError:
                    raise ValueError("Couldn't load %s password algorithm "
                                     "library" % name)
                return module
            raise ValueError("Hasher '%s' doesn't specify a library attribute" %
                             self.__class__)
    
        def salt(self):
            """
            Generates a cryptographically secure nonce salt in ascii
            """
            return get_random_string()
    
        def verify(self, password, encoded):
            """
            Checks if the given password is correct
            """
            raise NotImplementedError()
    
        def encode(self, password, salt):
            """
            Creates an encoded database value
    
            The result is normally formatted as "algorithm$salt$hash" and
            must be fewer than 128 characters.
            """
            raise NotImplementedError()
    
        def safe_summary(self, encoded):
            """
            Returns a summary of safe values
    
            The result is a dictionary and will be used where the password field
            must be displayed to construct a safe representation of the password.
            """
            raise NotImplementedError()

正如你看到那样，你必须创建下面的方法 ``verify()`` , ``encode()`` 以及 ``safe_summary()`` 。

Django 是使用 PBKDF 2算法与10,000次的迭代使得它不那么容易被暴力破解法轻易攻破。密码用下面的格式储存：

    algorithm$number of iterations$salt$password hash”

在这个例子中，Django 的 PBKDF2 使用 Base64 来加密。

::
    
    class PBKDF2PasswordHasher(BasePasswordHasher):
        """
        Secure password hashing using the PBKDF2 algorithm (recommended)
    
        Configured to use PBKDF2 + HMAC + SHA256 with 10000 iterations.
        The result is a 64 byte binary string.  Iterations may be changed
        safely but you must rename the algorithm if you change SHA256.
        """
        algorithm = "pbkdf2_sha256"
        iterations = 10000
        digest = hashlib.sha256
    
        def encode(self, password, salt, iterations=None):
            assert password
            assert salt and '$' not in salt
            if not iterations:
                iterations = self.iterations
            hash = pbkdf2(password, salt, iterations, digest=self.digest)
            hash = hash.encode('base64').strip()
            return "%s$%d$%s$%s" % (self.algorithm, iterations, salt, hash)
    
        def verify(self, password, encoded):
            algorithm, iterations, salt, hash = encoded.split('$', 3)
            assert algorithm == self.algorithm
            encoded_2 = self.encode(password, salt, int(iterations))
            return constant_time_compare(encoded, encoded_2)
    
        def safe_summary(self, encoded):
            algorithm, iterations, salt, hash = encoded.split('$', 3)
            assert algorithm == self.algorithm
            return SortedDict([
                (_('algorithm'), algorithm),
                (_('iterations'), iterations),
                (_('salt'), mask_hash(salt)),
                (_('hash'), mask_hash(hash)),
            ])

你也许担心你需要安装一些东西，Django 则已经包含了一个算法 ``utils.crypto`` 。

::
    
    def pbkdf2(password, salt, iterations, dklen=0, digest=None):
        """
        Implements PBKDF2 as defined in RFC 2898, section 5.2
    
        HMAC+SHA256 is used as the default pseudo random function.
    
        Right now 10,000 iterations is the recommended default which takes
        100ms on a 2.2Ghz Core 2 Duo.  This is probably the bare minimum
        for security given 1000 iterations was recommended in 2001. This
        code is very well optimized for CPython and is only four times
        slower than openssl's implementation.
        """
        assert iterations > 0
        if not digest:
            digest = hashlib.sha256
        hlen = digest().digest_size
        if not dklen:
            dklen = hlen
        if dklen > (2 ** 32 - 1) * hlen:
            raise OverflowError('dklen too big')
        l = -(-dklen // hlen)
        r = dklen - (l - 1) * hlen
    
        hex_format_string = "%%0%ix" % (hlen * 2)
    
        def F(i):
            def U():
                u = salt + struct.pack('>I', i)
                for j in xrange(int(iterations)):
                    u = _fast_hmac(password, u, digest).digest()
                    yield _bin_to_long(u)
            return _long_to_bin(reduce(operator.xor, U()), hex_format_string)
    
        T = [F(x) for x in range(1, l + 1)]
        return ''.join(T[:-1]) + T[-1][:r]

同时也提供了 ``bcrypt`` ，但是你需要先安装 ``py-bcryptor`` 。

我发现了一些很有趣的东西： Django doc 中的引文与 Django Source Code 所写的内容是相互矛盾的..

Django Doc

    本来Django使用PBKDF2算法和SHA256哈希算法，这是NIST推荐的一种把密码加密延长的方法。这对于大多数的用户足够了： 它非常安全，需要超级多的计算时间才能破解。

Django Source Code

    目前，2.2Ghz Core 2 Duo计算10,000次的迭代仅需要100毫秒，因此默认推荐使用此方法。在2001的推荐中这大概是耗时最少的算法了.这些代码为CPytoon做了很好的优化，仅仅比openssl慢4倍.

呃。。非常安全 VS 非常省时，我想知道我应该选哪个。
