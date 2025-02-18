from tkinter import *
from tkinter import filedialog
import threading
from PIL import Image
import yt_dlp
import customtkinter

customtkinter.set_appearance_mode("System")
customtkinter.set_default_color_theme("blue")

app = customtkinter.CTk()
app.geometry("630x320")
app.title("YouTube Downloader by ELHALOUI")
try:
    logo = customtkinter.CTkImage(light_image=Image.open("youtube_logo.png"), size=(100, 100))
    logo_label = customtkinter.CTkLabel(app, image=logo, text="")
    logo_label.place(relx=0.05, rely=0.2, anchor="w")
except Exception as e:
    print(f"Error loading logo: {e}")
try:
    logo1 = customtkinter.CTkImage(light_image=Image.open("lionnoir.jpg"), size=(100, 100))
    logo_label1 = customtkinter.CTkLabel(app, image=logo1, text="")
    logo_label1.place(relx=0.8, rely=0.2, anchor="w")
except Exception as e:
    print(f"Error loading logo: {e}")
def browse_folder():
    folder_selected = filedialog.askdirectory()
    folder_var.set(folder_selected)

def get_video_title(video_url):
    try:
        ydl_opts = {}
        with yt_dlp.YoutubeDL(ydl_opts) as ydl:
            info = ydl.extract_info(video_url, download=False)
            title = info.get('title', 'Unknown Title')
            return title

    except Exception as e:
        print(f"Error: {e}")
        return None
def update_progress(percent):
    Percentage.configure(text=f"Progress: {percent:.2f}%")
    progressbar.set(percent / 100)
    app.update_idletasks()  # Refresh UI
def progress_hook(d):
    if d['status'] == 'downloading':
        downloaded = d.get('downloaded_bytes', 0)
        total = d.get('total_bytes', None)  # Some videos might not have total_bytes

        if total and total > 0:
            percent = (downloaded / total) * 100
            app.after(100, lambda: update_progress(percent))  # Ensure UI updates

    elif d['status'] == 'finished':
        app.after(100, lambda: Percentage.configure(text="100%", text_color="green"))
        app.after(100, lambda: progressbar.set(1.0))  # Set to full

def start_download():
    try:
        video_link = url_entry.get()
        download_folder = folder_var.get()

        if not video_link:
            status_label.configure(text="Please enter a YouTube URL!", text_color="red")
            return

        if not download_folder:
            status_label.configure(text="Please select a download folder!", text_color="red")
            return

        ydl_opts = {
            'outtmpl': f"{download_folder}/%(title)s.%(ext)s",
            'progress_hooks': [progress_hook],
            'format': '18',
        }

        threading.Thread(target=lambda: download_video(video_link, ydl_opts), daemon=True).start()
        title = get_video_title(video_link)
        title_label.configure(text=title)
        status_label.configure(text="Download Completed", text_color="green")

    except Exception as e:
        status_label.configure(text=f"Download Failed! {str(e)}", text_color="red")

def download_video(video_url, ydl_opts):
    with yt_dlp.YoutubeDL(ydl_opts) as ydl:
        ydl.download([video_url])

# Welcome Label
welcome_label = customtkinter.CTkLabel(app, text="𝒲𝐸𝐿𝒞𝒪𝑀𝐸 𝒯𝒪 𝒴𝒪𝒰𝒯𝒰𝐵𝐸 𝒟𝒪𝒲𝒩𝐿𝒪𝒜𝒟𝐸𝑅", font=("", 22))
welcome_label.place(relx=0.5, rely=0.1, anchor="center")

powered_by_label = customtkinter.CTkLabel(app, text="𝒫𝑜𝓌𝑒𝓇𝑒𝒹 𝐵𝓎 𝐸𝐿𝐻𝒜𝐿𝒪𝒰𝐼", font=("", 15))
powered_by_label.place(relx=0.5, rely=0.2, anchor="center")

# URL Entry
url_label = customtkinter.CTkLabel(app, text="𝙀𝙉𝙏𝙀𝙍 𝙔𝙊𝙐𝙍 𝙔𝙊𝙐𝙏𝙐𝘽𝙀 𝙐𝙍𝙇", font=("", 18))
url_label.place(relx=0.02, rely=0.4, anchor="w")
url_var = StringVar()
url_entry = customtkinter.CTkEntry(app, width=310, height=25, textvariable=url_var)
url_entry.place(relx=0.5, rely=0.4, anchor="w")

# Folder Entry
folder_label = customtkinter.CTkLabel(app, text="𝙀𝙉𝙏𝙀𝙍 𝘿𝙊𝙒𝙉𝙇𝙊𝘼𝘿 𝙁𝙊𝙇𝘿𝙀𝙍", font=("", 18))
folder_label.place(relx=0.02, rely=0.5, anchor="w")
folder_var = StringVar()
folder_entry = customtkinter.CTkEntry(app, width=310, height=25, textvariable=folder_var)
folder_entry.place(relx=0.5, rely=0.5, anchor="w")

# Browse and Download Buttons
browse_button = customtkinter.CTkButton(app, text="BROWSE", width=80, command=browse_folder)
browse_button.place(relx=0.2, rely=0.6, anchor="w")

download_button = customtkinter.CTkButton(app, text="DOWNLOAD", width=120, command=start_download)
download_button.place(relx=0.6, rely=0.6, anchor="w")
title_label= customtkinter.CTkLabel(app, text="", text_color="green", font=("", 12))
title_label.place(relx=0.5, rely=0.75, anchor="center")
status_label = customtkinter.CTkLabel(app, text="", text_color="red", font=("", 12))
status_label.place(relx=0.5, rely=0.85, anchor="center")
Percentage=customtkinter.CTkLabel(app,text="0%")
Percentage.place(relx=0.5, rely=0.93, anchor="center")
progressbar= customtkinter.CTkProgressBar(app,width=400)
progressbar.set(0)
progressbar.place(relx=0.5, rely=0.97, anchor="center")

# Start the application
app.mainloop()
