# web-api-dynamic-cors
This will manage your CORS allowed origins in the web.config file. You get the capability of updating the allowing origins list without compiling and deploying your service each time the list changes.


# ICorsPolicyProvider
Instances that implement this interface will create a CORS policy based on a given http request. A CORS policy is a rule deciding how the CORS engine will process the CORS request. EnableCorsAttribute implements ICorsPolicyProvider. It turns all of your settings in the CORS attribute to CORS policy.

    'public class AllowCorsAttribute : Attribute, ICorsPolicyProvider
    {
        private const string keyCorsAllowHeader = "cors:allowHeaders";
        private const string keyCorsAllowMethod = "cors:allowMethods";
        private const string keyCorsAllowOrigin = "cors:allowOrigins";
        private const char seperator = ',';

        private CorsPolicy _policy;

        public Task<CorsPolicy> GetCorsPolicyAsync(
            HttpRequestMessage request,
            CancellationToken cancellationToken)
        {
            if (_policy == null)
            {
                var retval = new CorsPolicy();

                #region Headers
                var HeaderValue = ConfigurationManager.AppSettings[keyCorsAllowHeader];
                if (HeaderValue == "*")
                    retval.AllowAnyHeader = true;
                else
                {
                    retval.AllowAnyHeader = false;
                    if (!string.IsNullOrEmpty(HeaderValue))
                    {
                        foreach (var one in from v in HeaderValue.Split(seperator)
                                            where !string.IsNullOrEmpty(v)
                                            select v)
                        {
                            retval.Headers.Add(one);
                        }
                    }
                }
                #endregion

                #region Methods
                var MethodValue = ConfigurationManager.AppSettings[keyCorsAllowMethod];
                if (MethodValue == "*")
                    retval.AllowAnyMethod = true;
                else
                {
                    retval.AllowAnyMethod = false;
                    if (!string.IsNullOrEmpty(MethodValue))
                    {
                        foreach (var one in from v in MethodValue.Split(seperator)
                                            where !string.IsNullOrEmpty(v)
                                            select v)
                        {
                            retval.Methods.Add(one);
                        }
                    }
                }
                #endregion

                #region Origins
                var originValue = ConfigurationManager.AppSettings[keyCorsAllowOrigin];

                if (originValue == "*")
                    retval.AllowAnyOrigin = true;
                else
                {
                    retval.AllowAnyOrigin = false;
                    if (!string.IsNullOrEmpty(originValue))
                    {
                        foreach (var one in from v in originValue.Split(seperator)
                                            where !string.IsNullOrEmpty(v)
                                            select v)
                        {
                            retval.Origins.Add(one);
                        }
                    }
                }
                #endregion
                _policy = retval;
            }
            return Task.FromResult(_policy);
        }
    }'

AllowCors attribute derives from System.Attribute and implement ICorsPolicyProvider, therefore it will be picked up by AttributeBasedPolicyProviderFactory.

## Manage your CORS allowed origin in web.config
    
    '<appSettings>
      <add key="cors:allowHeaders" value="Origin,X-Requested-With,Content-Type,Accept,Accept-Language" />
      <add key="cors:allowMethods" value="GET,OPTIONS" />
      <add key="cors:allowOrigins" value="https://www.abc.ch,https://www.xyz.ch" />
    </appSettings>'

## How does it work
## Attributes your Controller

Replace the EnableCors attribute with AllowCors attribute for your Controller.

    '[AllowCors]
     public class DemoController : ApiController
     {
     }
    '
## How it works
Put your client origin in the web.config and start the service. Run your test client. It just works.
