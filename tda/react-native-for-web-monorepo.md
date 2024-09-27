# React Native for Web (MonoRepo)

Freebird App's CEO, [Chandrahasan Vantaku](https://www.linkedin.com/in/vantaku/), approached us with the task of developing a website clone of their existing mobile app(built using React Native). Recognizing the user preference for larger screens, they had to roll this out as soon as possible.

#### [React Native for Web](https://necolas.github.io/react-native-web/)

After a quick research of our options, we finalised on Reach Native for Web. It was a viable option as it was the fastest way to go to web without rewriting all the components. RN for Web is pure magic, if you didn't know, Twitter web is built on it. This gave us enough confidence that this a quite stable framework.

#### Monorepo Setup

We set up a Monorepo that contained both Mobile and Web code and configured webpack to build both web and app artefacts from the same components. We used NPM workspaces to set this up, this best suited our needs. The initial development was done by Ashish, then Saahith took over. I was there for some web issues and powered through the initial run.&#x20;

#### Challenges

* We had to tweak the webpack configuration so it uses RN for web to build for mobile. We also had to add overrides to support mobile.
* Once we were done with Webpack configuration, we had to carefully select cross-platform compatible NPM dependencies or map different dependencies based on the platform used.&#x20;
* Making React Navigation work on web was a total nightmare. We didn't think it was possible, but somehow we made it.
* To our bad luck, the mobile wasn't compiling post these changes anymore. Both the Android and iOS app did not work. So we had to sit with it to figure out why exactly it wasn't booting up.
* Then there were some very specific features that we had to ensure it works on both platforms. Like Export PDFs, Sharing loans.
* From a bottom nav on mobile, we created a left sidebar on web to use the space effectively.

There was a huge joy to us when we witnessed the web app running on localhost port 8080 after weeks of fixing compiler issues. We quickly deployed it on Firebase and informed the CEO. He was very happy about it.

They had their QA test the web app and came up with a whole bunch of bugs which we worked on.

#### Continuous Improvements

The CSS and User experience had to vary between mobile and web. So we had to tweak and write platform specific code wherever needed. One could argue that the components could have structured better. Very true, but the existing app code was too shitty to be salvaged. So this was inevitable.

#### Conclusion

I am not a React expert, but I understand why companies prefer to use MonoRepo setup. My opinion is unless its done from the start, its going to be a nightmare to maintain it. So unless its really needed, I wouldn't opt for it.

{% embed url="https://www.linkedin.com/posts/harishteens_reactnative-appdevelopment-webdevelopment-activity-7076931325303136256-OVMh?utm_source=share&utm_medium=member_desktop" %}
