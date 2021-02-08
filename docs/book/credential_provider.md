# Credential Provider

This feature is still in alpha and shouldn't be used in production environments.

As part of the cloud provider extraction, [KEP-2133](https://github.com/kubernetes/enhancements/pull/2151), proposed an extensible way to fetch credentials for pulling images. When kubelet needs credentials to fetch an image, it will now invoke a plugin based on the configuration provided by the cluster operator. Please see the original KEP for details.

We currently have the implementation for fetching ECR credentials. In order to use this new plugin, you'll have to

- Pass the folder where the binary is located as `--image-credential-provider-bin-dir` to the kubelet.
- Create a new `CredentialProviderConfig` and pass its location to the kubelet via `--image-credential-provider-config`.

Example config:
```
{
    "providers": [
        {
            "name": "ecr-credential-provider",
            "matchImages" : [
                "*.dkr.ecr.*.amazonaws.com,
                "*.dkr.ecr.*.amazonaws.com.cn,
            ],
            "apiVersion": "credentialprovider.kubelet.k8s.io/v1alpha1",
            "defaultCacheDuration": "0"
        }
    ]
}
```

Once you pass this config to the kubelet, everytime it needs to fetch an image that matches one of the "matchImages" patterns, it will invoke the "ecr-credential-provider" binary in the `--image-credential-provider-bin-dir` folder. In turn, the plugin will fetch the credentials for kubelet and send it back via stdio. Note that the name of the "provider" in your config has to match the name of the binary.