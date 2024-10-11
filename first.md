
Search
Write
Sign up

Sign in



How to Fix iOS Build Issues in Flutter Projects After Upgrading to macOS Sonoma and Xcode 16
Alexis Camué
Alexis Camué

·
Follow

4 min read
·
Sep 19, 2024




After upgrading to macOS Sonoma (Sequoia) and Xcode 16, many Flutter developers faced unexpected build issues in their iOS projects. In particular, errors related to BoringSSL-GRPC and other CocoaPods dependencies have become common. In this article, I’ll walk you through a step-by-step guide to fixing these problems, updating your Podfile, and cleaning caches to get your project running smoothly again.

This guide is particularly useful for Flutter developers who are working with iOS builds and have recently updated their macOS and Xcode versions.

Why Do These Errors Happen?
When Apple releases a new macOS or Xcode version, it often introduces changes that may not immediately be compatible with existing projects, especially ones that use libraries like BoringSSL-GRPC. CocoaPods dependencies, deployment targets, and compiler flags can all conflict with the new environment, causing frustrating build errors.

The most common errors that appear in the console include:

Unsupported compiler flags like -GCC_WARN_INHIBIT_ALL_WARNINGS
Incompatible iOS deployment targets
Dependency issues related to BoringSSL-GRPC
By making a few adjustments in your Podfile and cleaning up your project’s caches, you can resolve these problems quickly.

Step-by-Step Solution
Below are the steps that helped resolve the build issues in Xcode 16. This solution can be applied to any Flutter project that encounters these errors after updating your development environment.

Step 1: Modify Your Podfile
The Podfile is responsible for managing iOS dependencies in Flutter projects. Here’s the updated Podfile that fixed the build errors. Make sure you adjust the platform version if your project targets a different iOS version.

# Uncomment this line to define a global platform for your project
platform :ios, '12.0'
# CocoaPods analytics sends network stats synchronously affecting flutter build latency.
ENV['COCOAPODS_DISABLE_STATS'] = 'true'
project 'Runner', {
  'Debug' => :debug,
  'Profile' => :release,
  'Release' => :release,
}
def flutter_root
  begin
    generated_xcode_build_settings_path = File.expand_path(File.join('..', 'Flutter', 'Generated.xcconfig'), __FILE__)
    unless File.exist?(generated_xcode_build_settings_path)
      raise "#{generated_xcode_build_settings_path} must exist. If you're running pod install manually, make sure flutter pub get is executed first"
    end
    File.foreach(generated_xcode_build_settings_path) do |line|
      matches = line.match(/FLUTTER_ROOT\=(.*)/)
      return matches[1].strip if matches
    end
    raise "FLUTTER_ROOT not found in #{generated_xcode_build_settings_path}. Try deleting Generated.xcconfig, then run flutter pub get"
  rescue => e
    puts "Error: #{e.message}"
    raise
  end
end
require File.expand_path(File.join('packages', 'flutter_tools', 'bin', 'podhelper'), flutter_root)
flutter_ios_podfile_setup
target 'Runner' do
  use_frameworks!
  use_modular_headers!
  flutter_install_all_ios_pods File.dirname(File.realpath(__FILE__))
end
post_install do |installer|
  installer.pods_project.targets.each do |target|
    flutter_additional_ios_build_settings(target)
    # Fix issues with BoringSSL-GRPC
    if target.name == 'BoringSSL-GRPC'
      target.source_build_phase.files.each do |file|
        if file.settings && file.settings['COMPILER_FLAGS']
          flags = file.settings['COMPILER_FLAGS'].split
          # Remove the conflicting flag
          flags.reject! { |flag| flag == '-GCC_WARN_INHIBIT_ALL_WARNINGS' }
          file.settings['COMPILER_FLAGS'] = flags.join(' ')
        end
      end
    end
  end
end
Key Changes:

Set the minimum iOS platform version to '12.0'.
Removed conflicting compiler flags for BoringSSL-GRPC.
Included a safeguard to handle Flutter paths and ensure proper build settings are applied for iOS.
Step 2: Clean Pods and Caches
After updating your Podfile, the next step is to clean up your CocoaPods and Xcode caches to prevent conflicts between old versions and new configurations. Follow these commands:

# Navigate to the iOS directory
cd ios
# Remove the Pods directory
rm -rf Pods
# Clean Xcode's derived data (caches)
rm -rf ~/Library/Developer/Xcode/DerivedData/*
# Clean CocoaPods cache
pod cache clean --all
# Reinstall CocoaPods dependencies
pod install
Explanation:

rm -rf Pods: This removes the existing Pods folder to start fresh.
rm -rf ~/Library/Developer/Xcode/DerivedData/*: This cleans up any cached data that Xcode uses.
pod cache clean --all: This clears the CocoaPods cache to ensure there are no stale dependencies.
pod install: Reinstalls all Pods based on the updated Podfile.
Step 3: Rebuild the Project
After following the steps above, you can now rebuild your project with:

flutter clean
flutter pub get
flutter run
Running these commands will ensure that the Flutter project is rebuilt from scratch, applying all the fixes and resolving any lingering build errors.

Why This Works
Podfile Adjustments: The key to fixing these errors lies in updating the deployment target and removing problematic compiler flags that were introduced in the new Xcode version. By modifying the Podfile, we ensure that the project’s dependencies are aligned with the latest iOS and Xcode requirements.
Cleaning Caches: Outdated caches can cause unexpected issues when you switch Xcode or macOS versions. Cleaning these caches helps reset the environment and ensures a smooth rebuild of the project.
Reinstalling Dependencies: Once the environment is cleaned up, reinstalling the Pods ensures that the latest compatible versions of all dependencies are used, avoiding conflicts.
Conclusion
Upgrading to a new macOS and Xcode version often comes with unexpected build issues. However, with the right steps, you can quickly resolve these problems and continue building your Flutter projects for iOS. By modifying your Podfile and cleaning up your environment, you’ll be able to fix common errors related to BoringSSL-GRPC and CocoaPods in Xcode 16.

If you encounter similar issues after future updates, this guide will serve as a helpful reference to troubleshoot and fix the problem.

Feel free to save this process and apply it to other Flutter projects when needed!

Flutter
