require 'bundler'
Bundler.setup
require 'octokit'
require 'sanitize'
# @client = Octokit::Client.new(:access_token => ENV['GITHUB_API_TOKEN'])
@client = Octokit::Client.new

@milestones = @client.milestones(ENV['GITHUB_REPO'], {state: 'all'})

task :default do
  puts File.read('./_templates/header.txt')
  titles = create_array_of_milestone_titles
  titles.each do |title|
    ms = get_milestone_by_title(title)
    next unless ms
    puts "## [#{ms.title}](#{ms.url}) (#{ms.state})"
    puts "created_at: #{ms.created_at.to_s}  "
    streak = ""
    if ms.closed_at
      ((ms.closed_at - ms.created_at).to_i / 60 / 60 / 24).times {streak << "■"}
      puts "streak: #{streak}  "
      puts "closed_at: #{ms.closed_at.to_s if ms.closed_at}"
    end
    puts "<blockquote>"
    if ms.description.empty?
      puts "No Description."
    else
      puts ms.description
    end
    puts "</blockquote>"
    if ms.respond_to?(:number)
      issues = collect_issues_by_ms(ms.number)
      issues.each do |issue|
        str = "- "
        str <<  Sanitize.fragment(issue.title, Sanitize::Config::RESTRICTED)
        if issue.assignee
          str << " by #{issue.assignee.login}"
        end
        if issue.pull_request
          str << " at "
          str << "[PR-#{issue.number}](#{issue.pull_request.html_url}/files)"
        end
        puts str
        label_cols = issue.labels.map do |label|
          %Q{![#{label.name}](https://img.shields.io/badge/L-#{URI.encode(label.name)}-#{label.color}.svg)}
        end
        puts "    - #{label_cols.join(' ')}" unless label_cols.empty?
      end
    else
      puts "- Nothing comment"
    end

    puts ""
    puts "----"
  end
  puts File.read('./_templates/footer.txt')
end

def create_array_of_milestone_titles
  tags = @milestones.map do |a|
    begin
      Gem::Version.new(a.title.delete('v'))
    rescue
      next
    end
  end.sort.reverse

  tags.map {|tag| "v" + tag.to_s}
end

def get_milestone_by_title(title)
  @milestones.find {|m| m.title == title}
end

def collect_issues_by_ms(ms_number)
  @client.issues(ENV['GITHUB_REPO'], {state: 'all', milestone: ms_number})
end
