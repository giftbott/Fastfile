# If you want to automatically update fastlane if a new version is available:
#update_fastlane
#fastlane_version "2.38.0"

REQUIRED_XCODE_VERSION = "8.3.2"

default_platform :ios

### iOS Platform Setting ###
platform :ios do

  #################################################################### 
  def current_version_string
    build_number = get_build_number
    version_number = get_version_number
    "#{version_number}(#{build_number})"
  end

  def make_new_version_string(options)
    if options[:build]
      new_build_number = increment_build_number(build_number: options[:build])
    else
      new_build_number = increment_build_number
    end

    if options[:version]
      new_version_number = increment_version_number(version_number: options[:version])
    else
      new_version_number = get_version_number
    end

    "#{new_version_number}(#{new_build_number})"
  end

  def renew_certificates(isRelease=true)
    cert(
      output_path: "fastlane/certificates",
    )

    if isRelease
      pem(output_path: "fastlane/certificate")
    end

    UI.important("Release version? - #{isRelease}")
    sigh(
      adhoc: !isRelease,
      force: true,
      output_path: "fastlane/certificates",
    )

    config = isRelease ? "Release" : "ad-hoc"
    gym(
      scheme: ENV["SCHEME"],
      configuration: config,
      clean: true,
      silent: true,
      include_bitcode: false,
      output_name: ENV["SCHEME"] + "_#{current_version_string}_#{config}.ipa",
      output_directory: "fastlane/ipa",
    ) 
  end


  def common_release_process(options, type="release")
    ensure_xcode_version(version: REQUIRED_XCODE_VERSION)
    ensure_git_status_clean
    ensure_git_branch(
      branch: 'master'
    )

    release_version = make_new_version_string(options)

    clean_build_artifacts
    clear_derived_data
    renew_certificates

    commit_version_bump(
      message: "Version bumped to #{release_version}",
      xcodeproj: ENV["MAIN_PROJECT"],
    )

    if type == "beta"
      change_log = "Version: #{release_version}"
    else
      # http://git-scm.com/docs/pretty-formats
      change_log = changelog_from_git_commits(pretty: '%h %an %s')
    end

    change_log
  end


  def notify_to_slack(message, success='true', channel='#logs')
    slack(
      message: message,
      success: success,
      channel: channel,
      slack_url: ENV["SLACK_URL"]
    )
  end


  #################################################################### 
  before_all do

  end


  #################################################################### 
  #  Make Ad-Hoc ipa
  #################################################################### 
  lane :adhoc do |options|
    make_new_version_string(options)
    renew_certificates(false)
    notify_to_slack("Successfully created ad-hoc ipa. ver.")
  end


  #################################################################### 
  #  Deliver metadata to Itunesconnect
  #################################################################### 
  lane :metadata do |options|
    app_version = options[:appversion]
    if app_version
      deliver(
        force: true,
        skip_screenshots: true,
        automatic_release: true,
        app_version: app_version,
        submit_for_review: false,
      )
    else
      deliver(
        force: true,
        skip_screenshots: true,
        automatic_release: true,
        submit_for_review: false,
      )
    end
  end


  #################################################################### 
  #  Submit App for Review 
  #################################################################### 
  lane :submit_review do
    deliver(
      force: true,
      skip_metadata: true,
      skip_screenshots: true,
      automatic_release: true,
      submit_for_review: true,
    )
  end


  #################################################################### 
  #  Take Screenshots
  #################################################################### 
  lane :screenshot do |options|
    clear_derived_data
    snapshot
    frameit
    deliver
  end


  #################################################################### 
  #  Upload App For Testflight
  #################################################################### 
  lane :beta do |options|
    change_log = common_release_process(options, "beta")

    if options[:update_metadata]
      metadata   #call metadata method to do deliver action
      UI.important("Updated metadata")
    end

    testflight(
      changelog: change_log,
      skip_submission: true,
      distribute_external: false
    )

    notify_to_slack("New Production version pushed to TestFlight.")
    rocket
  end

  #################################################################### 
  #  Upload App For Release
  #################################################################### 
  lane :release do |options|
    change_log = common_release_process(options)
    #match(type: "appstore")
    #crashlytics

    metadata   #call metadata method to do deliver action
    UI.important("Updated metadata")

    add_git_tag(
      prefix: "v",
      build_number: get_version_number
    )

    push_to_git_remote(
      remote: "origin",
      local_branch: "master",
      remote_branch: "master",
    )

    clean_build_artifacts

    message = "Successfully deployed new App Update. ver - #{current_version_string}"
    notify_to_slack(message)
    rocket
  end


  #################################################################### 
  after_all do |lane|
    # This block is called, only if the executed lane was successful
  end


  #################################################################### 
  error do |lane, exception|
    message = "#{exception.message} with Version - #{current_version_string}"
    notify_to_slack(message, "false")
  end

end
## iOS platform end ##
