# ARGOCD DEMO

This repository is meant to test ARGOCD features. <br>
Integration has been done with `Github Application`. <br>
Ensure that you have `kubectl` and `argocd` cli. <br>

If you need to install `argocd` cli, here is the documentation : https://argo-cd.readthedocs.io/en/stable/cli_installation/ <br>
If you need to install `kubectl` cli, here is the documentation : https://kubernetes.io/docs/tasks/tools/install-kubectl/ <br>

If you need a kubernetes cluster, check out my repository : https://github.com/adriamanu/kube-ansible <br>

## Github Application Credentials
Here is a **ruby** script that generate the command for `argocd` cli in order to add your private repository thanks to your github application.

```ruby
require 'jwt'  # https://rubygems.org/gems/jwt
require 'openssl'
require 'net/http'
require 'json'

FILEPATH = "PATH_TO_YOUR_APPLICATION_PRIVATE_KEY"
APPLICATION_ID = "YOUR_APPLICATION_ID"
REPO = "YOUR_PRIVATE_REPO"

# Private key contents
private_pem = File.read(FILEPATH)
private_key = OpenSSL::PKey::RSA.new(private_pem)

# Generate the JWT
payload = {
  # issued at time, 60 seconds in the past to allow for clock drift
  iat: Time.now.to_i - 60,
  # JWT expiration time (10 minute maximum)
  exp: Time.now.to_i + (10 * 60),
  # GitHub App's identifier
  iss: APPLICATION_ID
}

jwt = JWT.encode(payload, private_key, "RS256")

res = Net::HTTP.get_response(URI('https://api.github.com/app/installations'),
  { 'Authorization' => "Bearer #{jwt}",
    "Accept" => "application/vnd.github.v3+json"
  }
)

data = JSON.parse(res.body)

# Print argo command
puts "argocd repo add " + REPO + " --github-app-id " + APPLICATION_ID + " --github-app-private-key-path " + FILEPATH + " --github-app-installation-id " + data[0]["id"].to_s
```

## Create Argo Project and Application
```bash
kubectl apply -f application.yaml -f project.yaml
```
