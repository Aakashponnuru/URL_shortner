#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <curl/curl.h>
#include <json-c/json.h>

// Bitly access token (replace with your actual token)
#define BITLY_ACCESS_TOKEN "c5622423eb0bbab035c95a80718ca8b1c80bbc57"

// Bitly API endpoint for shortening URLs
#define BITLY_API_URL "https://api-ssl.bitly.com/v4/shorten"

// File to store the mapping between short and long URLs
#define MAPPING_FILE "url_mappings.txt"

// Structure to store short and long URLs
struct UrlMapping {
    char shortUrl[256];
    char longUrl[256];
};

// Callback function for handling the response
size_t write_callback(void *contents, size_t size, size_t nmemb, void *userp) {
    // Parse JSON response
    struct json_object *root, *id, *longUrl;
    root = json_tokener_parse((char *)contents);

    if (root != NULL) {
        // Extract short and long URLs
        json_object_object_get_ex(root, "id", &id);
        json_object_object_get_ex(root, "long_url", &longUrl);

        const char *shortUrlStr = json_object_get_string(id);
        const char *longUrlStr = json_object_get_string(longUrl);

        // Open the file in append mode
        FILE *file = fopen(MAPPING_FILE, "a");

        if (file != NULL) {
            // Write short and long URLs to the file
            fprintf(file, "Short URL: https://%s\nLong URL: %s\n\n", shortUrlStr, longUrlStr);

            // Close the file
            fclose(file);
        }

        // Print the short and long URLs
        printf("Short URL: %s\nLong URL: %s\n\n", shortUrlStr, longUrlStr);

        // Release JSON object
        json_object_put(root);
    }

    return size * nmemb;
}

int main(int argc, char * argv[]) {
    // Initialize cURL
    CURL *curl = curl_easy_init();

    if (argc != 2)
        printf("usage %s <to be shortened url>\n", argv[0], argv[1]);

    if (curl) {
        // Set the URL to request
        const char *url = argv[1];
        curl_easy_setopt(curl, CURLOPT_URL, BITLY_API_URL);

        // Set the HTTP POST method
        curl_easy_setopt(curl, CURLOPT_POST, 1L);

        // Set the request body with the long URL
        char post_fields[256];
        snprintf(post_fields, sizeof(post_fields), "{\"long_url\": \"%s\"}", url);
        curl_easy_setopt(curl, CURLOPT_POSTFIELDS, post_fields);

        // Set the "Authorization" header with the access token
        struct curl_slist *headers = NULL;
        char auth_header[256];
        snprintf(auth_header, sizeof(auth_header), "Authorization: Bearer %s", BITLY_ACCESS_TOKEN);
        headers = curl_slist_append(headers, auth_header);
        headers = curl_slist_append(headers, "Content-Type: application/json");
        curl_easy_setopt(curl, CURLOPT_HTTPHEADER, headers);

        // Set the callback function to handle the response
        curl_easy_setopt(curl, CURLOPT_WRITEFUNCTION, write_callback);

        // Perform the request
        CURLcode res = curl_easy_perform(curl);

        // Check for errors
        if (res != CURLE_OK) {
            fprintf(stderr, "cURL request failed: %s\n", curl_easy_strerror(res));
        }

        // Clean up
        curl_easy_cleanup(curl);
        curl_slist_free_all(headers);
    }

    return 0;
}
