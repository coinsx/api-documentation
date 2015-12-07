
```ruby
require 'uri'
require 'net/http'
require 'time'
require 'securerandom'
require 'base64'

uri = URI.parse("https://sandbox.magnr.com")
http = Net::HTTP.new(uri.host, uri.port)
http.use_ssl = true

priv_key = "YOUR_PRIVATE_KEY"
pub_key = "YOUR_PUBLIC_KEY"

tonce = Time.now.to_i * 1000

data = "some json or empty string"
signature = Base64.strict_encode64(OpenSSL::HMAC.digest(OpenSSL::Digest.new('sha512'), priv_key, tonce.to_s+pub_key+data))
auth = Base64.strict_encode64("#{pub_key}:#{signature}")

request = Net::HTTP::Get.new("/api/v1/<some end point>/")
request.add_field('Content-Type', 'application/json')
request.add_field('Authorization', "Basic #{auth}")
request.add_field('Tonce', tonce)

response = http.request(request)
```