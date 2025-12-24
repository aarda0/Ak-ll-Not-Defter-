import flet as ft
import os
from datetime import datetime

# Klasör ayarları
NOTES_DIR = "ders_notlari"
if not os.path.exists(NOTES_DIR):
    os.makedirs(NOTES_DIR)

def main(page: ft.Page):
    page.title = "Ultimate Not Defteri v3.0"
    page.theme_mode = ft.ThemeMode.DARK
    page.window_width = 500
    page.window_height = 850
    page.padding = 0
    page.bgcolor = ft.Colors.GREY_900  # Background rengi
    page.fonts = {
        "Poppins": "https://github.com/google/fonts/raw/main/ofl/poppins/Poppins-Regular.ttf"
    }
    page.theme = ft.Theme(font_family="Poppins")

    # --- YARDIMCI FONKSİYONLAR ---
    def get_file_path(ders_adi):
        return os.path.join(NOTES_DIR, f"{ders_adi}.txt")

    def save_note(e, ders_adi, label):
        content = e.control.value
        with open(get_file_path(ders_adi), "w", encoding="utf-8") as f:
            f.write(content)
        label.value = f"Son Düzenleme: {datetime.now().strftime('%H:%M:%S')}"
        label.update()

    def export_pdf(e, ders_adi):
        # Basit bir dışa aktarma simülasyonu (txt kopyası)
        path = get_file_path(ders_adi)
        if os.path.exists(path):
            page.snack_bar = ft.SnackBar(ft.Text(f"{ders_adi} PDF olarak hazırlandı (Simüle edildi)"), bgcolor="green")
            page.snack_bar.open = True
            page.update()

    # --- ANA BİLEŞENLER ---
    tabs = ft.Tabs(
        selected_index=0,
        animation_duration=400,
        expand=True,
        scrollable=True, # Dersler çoksa kaydırılabilir tab bar
    )

    def build_tab(ders_adi):
        path = get_file_path(ders_adi)
        content = ""
        if os.path.exists(path):
            with open(path, "r", encoding="utf-8") as f:
                content = f.read()

        update_label = ft.Text("Düzenleme bekleniyor...", size=11, italic=True, color=ft.Colors.BLUE_200)

        return ft.Tab(
            tab_content=ft.Row([ft.Icon(ft.Icons.BOOK), ft.Text(ders_adi)]),
            content=ft.Container(
                content=ft.Column([
                    # Üst Bilgi Kartı
                    ft.Card(
                        content=ft.Container(
                            content=ft.Row([
                                ft.Column([
                                    ft.Text(ders_adi.upper(), size=22, weight="bold", color=ft.Colors.AMBER_400),
                                    update_label,
                                ], expand=True),
                                ft.IconButton(ft.Icons.PICTURE_AS_PDF, tooltip="Dışa Aktar", on_click=lambda e: export_pdf(e, ders_adi)),
                                ft.IconButton(ft.Icons.DELETE_FOREVER, icon_color="red", on_click=lambda _: delete_ders(ders_adi)),
                            ]),
                            padding=15
                        ),
                        elevation=5
                    ),
                    # Not Alanı
                    ft.Container(
                        content=ft.TextField(
                            value=content,
                            multiline=True,
                            expand=True,
                            border=ft.InputBorder.NONE,
                            hint_text="Buraya notlarını girmeye başla...",
                            on_change=lambda e: save_note(e, ders_adi, update_label),
                            text_size=16,
                        ),
                        bgcolor=ft.Colors.with_opacity(0.05, ft.Colors.WHITE),
                        padding=15,
                        border_radius=10,
                        expand=True
                    )
                ]),
                padding=20,
            )
        )

    def refresh_tabs(search_term=""):
        files = [f.replace(".txt", "") for f in os.listdir(NOTES_DIR) if f.endswith(".txt")]
        if not files and not search_term:
            files = ["Genel"]
            with open(get_file_path("Genel"), "w") as f: f.write("")
        
        # Arama filtresi
        filtered_files = [f for f in files if search_term.lower() in f.lower()]
        
        tabs.tabs = [build_tab(d) for d in filtered_files]
        page.update()

    # --- ARAMA VE EKLEME ---
    def on_search(e):
        refresh_tabs(e.control.value)

    search_field = ft.TextField(
        hint_text="Derslerde ara...",
        prefix_icon=ft.Icons.SEARCH,
        on_change=on_search,
        height=40,
        text_size=14,
        content_padding=10,
        border_radius=20,
        bgcolor=ft.Colors.with_opacity(0.1, ft.Colors.WHITE)
    )

    def add_ders(e):
        if not new_ders_input.value: return
        with open(get_file_path(new_ders_input.value), "w", encoding="utf-8") as f:
            f.write("")
        new_ders_input.value = ""
        dialog.open = False
        refresh_tabs()

    def delete_ders(ders_adi):
        os.remove(get_file_path(ders_adi))
        refresh_tabs()

    new_ders_input = ft.TextField(label="Ders/Konu Başlığı", autofocus=True)
    dialog = ft.AlertDialog(
        title=ft.Text("Yeni Bölüm Ekle"),
        content=new_ders_input,
        actions=[
            ft.TextButton("Kapat", on_click=lambda _: setattr(dialog, "open", False) or page.update()),
            ft.ElevatedButton("Oluştur", on_click=add_ders, bgcolor=ft.Colors.BLUE_700, color="white"),
        ],
    )
    page.overlay.append(dialog)
    page.update()  # Overlay güncellemesi için gerekli

    # --- UI GÖRÜNÜM ---
    page.appbar = ft.AppBar(
        title=ft.Text("STUDY NOTE PRO", weight="bold", size=20),
        center_title=False,
        bgcolor=ft.Colors.BLUE_GREY_900,
        actions=[
            ft.Container(content=search_field, width=200, padding=5),
            ft.IconButton(ft.Icons.ADD_BOX_ROUNDED, icon_size=30, on_click=lambda _: setattr(dialog, "open", True) or page.update()),
            ft.PopupMenuButton(
                items=[
                    ft.PopupMenuItem(icon=ft.Icons.PALETTE, text="Tema Değiştir", on_click=lambda _: toggle_theme()),
                    ft.PopupMenuItem(icon=ft.Icons.INFO, text="Hakkında")
                ]
            )
        ]
    )

    def toggle_theme():
        page.theme_mode = ft.ThemeMode.LIGHT if page.theme_mode == ft.ThemeMode.DARK else ft.ThemeMode.DARK
        page.update()

    page.add(tabs)
    refresh_tabs()

ft.app(target=main)
