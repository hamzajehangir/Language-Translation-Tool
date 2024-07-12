# Language-Translation-Tool

The language used in this code is C# (C Sharp). It is a modern, object-oriented programming language developed

using System;
using System.Net.Http;
using System.Text;
using Newtonsoft.Json;

namespace LanguageTranslator
{
    class Translator
    {
        private readonly string _apiKey;
        private readonly string _apiEndpoint;
        private readonly string _apiLocation;

        public Translator(string apiKey, string apiEndpoint, string apiLocation)
        {
            _apiKey = apiKey;
            _apiEndpoint = apiEndpoint;
            _apiLocation = apiLocation;
        }

        public async Task<string> TranslateText(string text, string fromLanguage, string toLanguage)
        {
            string route = $"/translate?api-version=3.0&from={fromLanguage}&to={toLanguage}";
            object[] body = new object[] { new { Text = text } };
            var requestBody = JsonConvert.SerializeObject(body);

            using (var client = new HttpClient())
            using (var request = new HttpRequestMessage())
            {
                request.Method = HttpMethod.Post;
                request.RequestUri = new Uri(_apiEndpoint + route);
                request.Content = new StringContent(requestBody, Encoding.UTF8, "application/json");
                request.Headers.Add("Ocp-Apim-Subscription-Key", _apiKey);
                request.Headers.Add("Ocp-Apim-Subscription-Region", _apiLocation);

                HttpResponseMessage response = await client.SendAsync(request).ConfigureAwait(false);
                response.EnsureSuccessStatusCode();
                string responseBody = await response.Content.ReadAsStringAsync();

                var jsonData = JsonConvert.DeserializeObject<TranslationResponse>(responseBody);
                return jsonData.Translations[0].Text;
            }
        }
    }

    public class TranslationResponse
    {
        public Translation[] Translations { get; set; }
    }

    public class Translation
    {
        public string Text { get; set; }
    }

    class Program
    {
        static async Task Main(string[] args)
        {
            string apiKey = "<your-translator-key>";
            string apiEndpoint = "https://api.cognitive.microsofttranslator.com";
            string apiLocation = "<YOUR-RESOURCE-LOCATION>";

            Translator translator = new Translator(apiKey, apiEndpoint, apiLocation);

            string textToTranslate = "I would really like to drive your car around the block a few times!";
            string fromLanguage = "en";
            string toLanguage = "fr";

            string translatedText = await translator.TranslateText(textToTranslate, fromLanguage, toLanguage);

            Console.WriteLine($"Translated text: {translatedText}");
        }
    }
}


