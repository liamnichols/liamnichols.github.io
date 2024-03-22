---
layout: post
title: Privacy Manifests
keywords: xcode15, apple, privacy, xcprivacy, privacy manifest, feedback
---

During WWDC 2023, Apple announced the [introduction of Privacy Manifests](https://developer.apple.com/videos/play/wwdc2023/10060). The motivations behind Privacy Manifests are very much justified, but the way that this ecosystem-wide change is being forced upon developers with barely any support is frustrating to say the least.

I did originally write a lengthy rant about my experience, but I figure that it wasn't entirely productive. Instead, if like me you are also frustrated and would like Apple to reconsider their approach, please submit a Feedback Request letting them know.

If you've never submitted a Feedback Request before, or you want help with what exactly you should write, read on. I'll only take a couple of minutes!

1. Head to [**feedbackassistant.apple.com**](https://feedbackassistant.apple.com/) and Sign In with your developer account
2. Click the [**Compose New Feedback**](https://feedbackassistant.apple.com/new-form-response) button at the top of the page
3. Select **Developer Tools & Resources**
4. Enter the details below (or feel free to get creative yourself):

    **Please provide a descriptive title for your feedback:**

    <p class="highlight" style="padding: 16px">
      Please better support developers with the upcoming Privacy Manifest requirements
    </p>

    **Which area are you seeing an issue with?**

    <p class="highlight" style="padding: 16px">
      Something else not on this list
    </p>

    **What type of feedback are you reporting?**

    <p class="highlight" style="padding: 16px">
      Suggestion
    </p>

    **Please describe the issue and what steps we can take to reproduce it**

    <div class="highlight" style="padding: 16px">
      <p>
        For developers like me, the rollout of the Privacy Manifest requirements has been difficult and frustrating.
      </p>

      <p>
        Please stop App Store Connect from reporting warnings about my Privacy Manifest and postpone any enforcement relating to the Privacy Manifest until you are able to better guide developers to implement the required changes. To help guide developers across the ecosystem, I suggest the following actions:
      </p>

      <p>
        1. Complete the integration within Xcode and Swift Package Manager - Currently, manifests provided by third party developers have no way of being incorporated into the App's main PrivacyManifest.xcprivacy file when dependencies are statically linked to a target. It seems that the only workaround is for the developer to manually copy the contents of the third party Privacy Manifest into their own which is not sustainable.
      </p>

      <p>
        2. Provide a tool or documentation to locally automate the analysis of required API usage - The warning emails from App Store Connect demonstrate that this is possible, so please provide a tool to assist developers in building Privacy Manifests instead of having to manually audit code against the documentation (which is subject to change). This tool should be usable by both app developers and third-party SDK developers.
      </p>

      <p>
        3. Provide more assistance to third-party dependency managers - CocoaPods is still used by hundreds of thousands of developers, and it is critical that this tool properly supports Privacy Manifests. Please make more effort to support CocoaPods and other third-party tools by offering support via Developer Relations and/or sponsoring any required work.
      </p>

      <p>
        4. Provide more documentation - Ensuring that Privacy Manifests are properly supplied and configured for first and third-party code is still not simple enough. Please provide more documentation, technical notes, and troubleshooting guides for ensuring that Privacy Manifests are set up correctly. There is currently no good resource on the internet for this, and developers are finding themselves overwhelmed having to figure it out against a deadline.
      </p>

      <p>
        I (and I am sure many others) want to help respect the privacy of my users and provide them with full transparency, but with the current friction around Privacy Manifests, it is hard to do so, and I fear that by upsetting developers, it will have a negative effect on the overall goals of this change. For Privacy Manifests and required API usage declarations to be meaningful and effective, the process needs to be well-understood and frictionless, and I hope that these suggestions can be applied in order to help achieve the goals.
      </p>
    </div>

5. Click **Submit** and hope for the best

---

_Updated on 2024-03-22 at 15:44 CET: Added additional request to the list of actions._
