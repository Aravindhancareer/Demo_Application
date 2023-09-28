# Get the cluster IP of the Kubernetes pod.
$clusterIP = Get-Item "Kubernetes:\services\my-service" | Select-Object status.loadBalancer.ingress[0].ip

# Get the path to the .zip file.
$zipFilePath = "C:\path\to\my.zip"

# Generate a UUID for the `clientIdentifier` header.
$clientIdentifier = [System.Guid]::NewGuid().ToString()

# Construct the HTTP request body.
$requestBody = @{
    "appPackage" = [System.IO.File]::OpenRead($zipFilePath)
}

# Create the HTTP client.
$client = New-Object System.Net.Http.HttpClient

# Set the request method and URI.
$client.Method = "POST"
$client.RequestUri = "http://$clusterIP:8080/container/{containerid}/apps"

# Add the `containerId` and `clientIdentifier` headers to the request.
$client.DefaultRequestHeaders.Add("containerId", "my-container-id")
$client.DefaultRequestHeaders.Add("clientIdentifier", $clientIdentifier)

# Add the request body to the request.
$client.DefaultRequestHeaders.Add("Content-Type", "multipart/form-data")
$client.RequestUri.Content = $requestBody

# Send the request and get the response.
$response = $client.SendAsync().Result

# Check the response status code.
if ($response.StatusCode -eq HttpStatusCode.OK) {
    # The app was created successfully.
    Write-Host "The app was created successfully."
} else {
    # An error occurred while creating the app.
    Write-Host "An error occurred while creating the app: $response.StatusCode"
}
