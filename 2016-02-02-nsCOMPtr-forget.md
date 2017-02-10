# nsCOMPtr `forget`
## bug 1335526 co-inciding with my landing of bug 1073952

This code was modified from nsScriptSecurityManager::GetChannelResultPrincipal in file [caps/nsScriptSecurityManager.cpp](http://searchfox.org/mozilla-central/diff/b62bca9af3dc9bb1c791409e0301ae9c877981df/caps/nsScriptSecurityManager.cpp#297) due to [bug 1335526](https://bugzilla.mozilla.org/show_bug.cgi?id=1335526)
``` diff
-             RefPtr<nsNullPrincipal> prin;
-             if (loadInfo->LoadingPrincipal()) {
-               prin =
-                 nsNullPrincipal::CreateWithInheritedAttributes(loadInfo->LoadingPrincipal());
-             } else {
-               OriginAttributes attrs;
-               loadInfo->GetOriginAttributes(&attrs);
-               attrs.StripAttributes(OriginAttributes::STRIP_ADDON_ID);
-               prin = nsNullPrincipal::Create(attrs);
-             }
-             prin.forget(aPrincipal);
-             return NS_OK;
+           MOZ_ALWAYS_TRUE(NS_SUCCEEDED(loadInfo->GetSandboxedLoadingPrincipal(aPrincipal)));
+           MOZ_ASSERT(*aPrincipal);
+           return NS_OK;
```
It was re-added in Loadinfo.cpp like so:
``` cpp
NS_IMETHODIMP
LoadInfo::GetSandboxedLoadingPrincipal(nsIPrincipal** aPrincipal)
{
  if (!(mSecurityFlags & nsILoadInfo::SEC_SANDBOXED)) {
    *aPrincipal = nullptr;
    return NS_OK;
  }

  if (!mSandboxedLoadingPrincipal) {
    if (mLoadingPrincipal) {
      mSandboxedLoadingPrincipal =
        nsNullPrincipal::CreateWithInheritedAttributes(mLoadingPrincipal);
    } else {
      OriginAttributes attrs(mOriginAttributes);
      attrs.StripAttributes(OriginAttributes::STRIP_ADDON_ID);
      mSandboxedLoadingPrincipal = nsNullPrincipal::Create(attrs);
    }
  }
  MOZ_ASSERT(mSandboxedLoadingPrincipal);

  nsCOMPtr<nsIPrincipal> copy(mSandboxedLoadingPrincipal);
  copy.forget(aPrincipal);
  return NS_OK;
}
```

What I learned (with some help from Miko):
* I don't really understand our smart pointers (yet)
* `newCOMPtr.forget(aParam)` makes `aParam` overwrite its original mRawPtr and take the one from `newCOMPtr`.


Or as the [source code](https://dxr.mozilla.org/mozilla-central/rev/1d025ac534a6333a8170a59a95a8a3673d4028ee/xpcom/base/nsCOMPtr.h#731) says:


```
// Set the target of aRhs to the value of mRawPtr and null out mRawPtr.
// Useful to avoid unnecessary AddRef/Release pairs with "out" parameters
// where aRhs bay be a T** or an I** where I is a base class of T.
template<typename I>
void forget(I** aRhs)
{
  NS_ASSERTION(aRhs, "Null pointer passed to forget!");
  NSCAP_LOG_RELEASE(this, mRawPtr);
  *aRhs = get();
  mRawPtr = nullptr;
}
```
