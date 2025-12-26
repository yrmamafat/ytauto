# main.py
import boto3
import openai
import requests
import logging
import pyttsx3
from googleapiclient.discovery import build
from googleapiclient.http import MediaFileUpload
from config import amazon_access_key, amazon_secret_key, amazon_associate_tag, openai_api_key, youtube_api_key, youtube_credentials_file

# Logging setup
logging.basicConfig(filename='automation_log.txt', level=logging.INFO)

# Initialize OpenAI API
openai.api_key = openai_api_key

# Initialize Amazon API client (using boto3)
def fetch_amazon_products(category, min_price, max_price, min_rating):
    try:
        client = boto3.client(
            'product-advertising-api', 
            region_name='us-west-1',
            aws_access_key_id=amazon_access_key,
            aws_secret_access_key=amazon_secret_key
        )
        response = client.search_products(
            category=category,
            min_price=min_price,
            max_price=max_price,
            min_review_rating=min_rating,
            associate_tag=amazon_associate_tag
        )
        return response['Items']
    except Exception as e:
        logging.error(f"Error fetching products: {e}")
        return []

# Function to generate product review script using OpenAI
def generate_review_script(product_name, product_category, price, rating, affiliate_link):
    prompt = f"""
    Write a product review script for {product_name} from Amazon. The product is {product_category} and is priced at {price}. 
    The product has a rating of {rating} stars on Amazon. The script should include an introduction, key features, pros, cons, 
    and a conclusion. Add a call-to-action to visit the product link: {affiliate_link}.
    """
    
    response = openai.Completion.create(
        engine="text-davinci-003",  
        prompt=prompt,
        max_tokens=500  
    )
    
    script = response.choices[0].text.strip()
    return script

# Function to generate voiceover using pyttsx3
def generate_voiceover(script):
    engine = pyttsx3.init()
    engine.save_to_file(script, 'voiceover.mp3')
    engine.runAndWait()

# Function to upload video to YouTube
def upload_video_to_youtube(video_file, title, description, tags, privacy_status="public"):
    youtube = build('youtube', 'v3', developerKey=youtube_api_key)
    
    request = youtube.videos().insert(
        part="snippet,status",
        body=dict(
            snippet=dict(
                title=title,
                description=description,
                tags=tags
            ),
            status=dict(
                privacyStatus=privacy_status
            )
        ),
        media_body=MediaFileUpload(video_file)
    )
    request.execute()

# Main script flow
if __name__ == "__main__":
    # Fetch products
    products = fetch_amazon_products("electronics", 50, 500, 4)

    # For each product, generate a script, voiceover, and upload video
    for product in products:
        product_name = product['Title']
        product_category = product['Category']
        price = product['Price']
        rating = product['Rating']
        affiliate_link = product['AffiliateLink']
        
        script = generate_review_script(product_name, product_category, price, rating, affiliate_link)
        generate_voiceover(script)
        
        # Here, integrate the video creation logic (using Synthesia, Pictory, etc.)
        video_file = "generated_video.mp4"  # Placeholder for the actual video generation process
        
        upload_video_to_youtube(video_file, product_name, script, ["affiliate", "product review", "Amazon"])
        
        logging.info(f"Uploaded video for {product_name} successfully.")
