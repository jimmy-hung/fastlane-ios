# This file contains the fastlane.tools configuration
# You can find the documentation at https://docs.fastlane.tools
#
# For a list of all available actions, check out
#
#     https://docs.fastlane.tools/actions
#
# For a list of all available plugins, check out
#
#     https://docs.fastlane.tools/plugins/available-plugins
#

# Uncomment the line if you want fastlane to automatically update itself
# update_fastlane

fastlane_version "2.62.0"

default_platform(:ios)

platform :ios do
  desc "Description of what the lane does"
##

  lane :archive_adhoc do |options|
    begin
      # 設定Keychain
      if !options[:skip_setup_circle_ci]
        setup_circle_ci
      end
      xcode_select("/Applications/Xcode#{ENV['XCODE_VERSION'].nil? ? "" : "-" + ENV['XCODE_VERSION']}.app")
      # 將 Pipeline ID 設定到build number
      increment_build_number(build_number: ENV['CI_PIPELINE_IID'])
      # 同步憑證
      match(type: "adhoc", clone_branch_directly: true, readonly: true)

      project = Xcodeproj::Project.open(Dir["../*.xcodeproj"].first)
      # 將 AppStore 替換成 AdHoc
      project.build_configurations.each do |config|
        unless config.build_settings['PROVISIONING_PROFILE_SPECIFIER'].nil?
          specifier = config.build_settings['PROVISIONING_PROFILE_SPECIFIER']
          config.build_settings['PROVISIONING_PROFILE_SPECIFIER'] = specifier.sub 'AppStore', 'AdHoc'
        end
      end
      project.targets.each do |target|
        target.build_configurations.each do |config|
          unless config.build_settings['PROVISIONING_PROFILE_SPECIFIER'].nil?
            specifier = config.build_settings['PROVISIONING_PROFILE_SPECIFIER']
            config.build_settings['PROVISIONING_PROFILE_SPECIFIER'] = specifier.sub 'AppStore', 'AdHoc'
          end
        end
      end
      project.save

      gym(scheme: ENV['XCODE_SCHEME'],
        export_method: "ad-hoc",
        skip_profile_detection: true,
        export_options: {
          compileBitcode: false
        }
      )
    ensure
      unless Helper.ci?
        UI.message("Not running on CI, skipping delete_keychain")
        next
      end
      delete_keychain(name: ENV['MATCH_KEYCHAIN_NAME'])
    end
  end

lane :archive_appstore do |options|
    begin
      if !options[:skip_setup_circle_ci]
        setup_circle_ci
      end
      xcode_select("/Applications/Xcode#{ENV['XCODE_VERSION'].nil? ? "" : "-" + ENV['XCODE_VERSION']}.app")
      increment_build_number(build_number: ENV['CI_PIPELINE_IID'])
      match(type: "appstore", clone_branch_directly: true, readonly: true)
      gym(scheme: ENV['XCODE_SCHEME'], skip_profile_detection: true) # Build your app - more options available
    ensure
      unless Helper.ci?
        UI.message("Not running on CI, skipping delete_keychain")
        next
      end
      delete_keychain(name: ENV['MATCH_KEYCHAIN_NAME'])
    end
  end
  
  desc "Submit a new Beta Build to Firebase"
  desc "This will also make sure the profile is up to date"
  lane :beta_firebase do |options|
    firebase_app_distribution(
      app: ENV['FIREBASE_APP'],
      firebase_cli_token: ENV['FIREBASE_CLI_TOKEN'],
      groups: ENV['FIREBASE_GROUPS']
    )
    unless ENV['GOOGLE_SERVICE_PLIST_PATH'].nil?
      upload_symbols_to_crashlytics(gsp_path: ENV["GOOGLE_SERVICE_PLIST_PATH"])
    end
  end

desc "Submit a new Beta Build to Apple TestFlight"
  desc "This will also make sure the profile is up to date"
  lane :beta_testflight do |options|
    begin
      if !options[:skip_setup_circle_ci]
        setup_circle_ci
      end
      xcode_select("/Applications/Xcode#{ENV['XCODE_VERSION'].nil? ? "" : "-" + ENV['XCODE_VERSION']}.app")
      pilot(skip_waiting_for_build_processing: true)
    ensure
      unless Helper.ci?
        UI.message("Not running on CI, skipping delete_keychain")
        next
      end
      delete_keychain(name: ENV['MATCH_KEYCHAIN_NAME'])
    end
  end

desc "Generate devices.txt"
  lane :add_device do
    sh('echo "Device ID	Device Name" >> ../devices.txt')
    sh('echo "A123456789012345678901234567890123456789	NAME1" >> ../devices.txt')
    sh('echo "B123456789012345678901234567890123456789	NAME2" >> ../devices.txt')
  end

  lane :register_a_device do
    register_devices(
      devices_file: "./devices.txt"
    )
    refresh_profiles
    sh('rm -f ../devices.txt')
  end

  # A helper lane for refreshing provisioning profiles.
  lane :refresh_profiles do
    match(
      type: "development",
      force: true,
      clone_branch_directly: true)
    match(
      type: "adhoc",
      force: true,
      clone_branch_directly: true)
  end


##
end
