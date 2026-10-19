---
this_file: WORK.md
---

# Framework loading fix

- Reproduced the reported WiFi Signal reinstall failure with the installed release.
- Confirmed the release workflow builds without the user-specific rpath added by the Makefile.
- Consulted Apple's dlopen/run-path documentation and the local dyld manual; search-path environment variables must be present when dyld starts.
- Added a startup helper using dyld fallback paths and one execv before any command side effects. Fallback paths avoid overriding the system Swift runtime with Setapp's bundled copy.
- Unit suite: 146 tests executed, one skipped because Setapp is installed, zero failures. The local SwiftPM test launcher cannot locate XCTest; built the test bundle with explicit Xcode XCTest include/library paths and ran Xcode's xctest directly. No system toolchain settings changed.
- Standalone integration: reproduces a missing @rpath dependency without the helper; loads the framework, sibling dylib, and transitive framework with it. Confirms paths/arguments with spaces and exactly one relaunch.
- Live debug test: SetappInterface loads; the subsequent XPC request is rejected with Setapp error -401010 / error.system-integrity-violated. The user confirms the signed-in GUI works, so this must not be described as an account sign-in failure.
- SwiftLint 0.61.0 passes with an explicit local SourceKit toolchain path. Changed files pass SwiftFormat 0.64.0; also formatted three pre-existing files rejected by its new multiline-conditional rule.
- No package dependencies added or changed.
- Apple Silicon release build passes. Local universal linking is blocked by missing x86_64 Swift compatibility archives in the installed toolchain. CI now explicitly builds and verifies both release architectures on macOS 15.
- Comparison build: used the upstream command entry point with an embedded Setapp framework rpath and no startup-helper invocation. It loads SetappInterface and receives the identical XPC -401010 rejection. This independently confirms that rejection is not introduced by the relaunch or dyld fallback paths.
- SwiftFormat 0.64.0 and SwiftLint 0.61.0 both pass across the repository after formatting the existing conditional bodies.
- Pending: publish the verified branch, open the upstream PR, and inspect CI release verification. Full live reinstall remains unverified because Setapp rejects the upstream client too.
