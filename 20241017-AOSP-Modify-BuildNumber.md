# AOSP Modify BuildNumber

## environment
* AOSP-13.0.0_r78

## step
* build/make/core/version_util.mk

```diff
HAS_BUILD_NUMBER := true
ifndef BUILD_NUMBER
  # BUILD_NUMBER should be set to the source control value that
  # represents the current state of the source code.  E.g., a
  # perforce changelist number or a git hash.  Can be an arbitrary string
  # (to allow for source control that uses something other than numbers),
  # but must be a single word and a valid file name.
  #
  # If no BUILD_NUMBER is set, create a useful "I am an engineering build
  # from this date/time" value.  Make it start with a non-digit so that
  # anyone trying to parse it as an integer will probably get "0".
--  BUILD_NUMBER := eng.$(shell echo $${BUILD_USERNAME:0:6}).$(shell $(DATE) +%Y%m%d.%H%M%S)
--  HAS_BUILD_NUMBER := false

++  BUILD_NUMBER := 10316532
++  HAS_BUILD_NUMBER := true
endif
```
* make installclean
* m -j$(nproc)
