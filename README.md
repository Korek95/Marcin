# .github/workflows/build.yml
name: Build Application

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: windows-latest
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: '3.9'
        
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install cx_Freeze pillow
        
    - name: Build executable
      run: python setup.py build
        
    - name: Create ZIP archive
      run: |
        cd build
        Compress-Archive -Path .\exe.win-amd64-3.9\* -DestinationPath ..\warehouse-app.zip
        
    - name: Upload artifact
      uses: actions/upload-artifact@v2
      with:
        name: warehouse-app
        path: warehouse-app.zip
        
    - name: Create Release
      id: create_release
      uses: actions/create-release@v1
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        tag_name: v${{ github.run_number }}
        release_name: Release v${{ github.run_number }}
        draft: false
        prerelease: false
        
    - name: Upload Release Asset
      uses: actions/upload-release-asset@v1
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        upload_url: ${{ steps.create_release.outputs.upload_url }}
        asset_path: ./warehouse-app.zip
        asset_name: warehouse-app.zip
        asset_content_type: application/zip
# setup.py
import sys
from cx_Freeze import setup, Executable

# Dependencies are automatically detected, but it might need fine tuning.
build_exe_options = {
    "packages": ["tkinter", "sqlite3", "PIL"],
    "excludes": ["tkinter.test", "unittest"],
    "include_files": [],
}

base = None
if sys.platform == "win32":
    base = "Win32GUI"

setup(
    name="System Magazynowy",
    version="1.0",
    description="System zarządzania magazynem",
    options={"build_exe": build_exe_options},
    executables=[Executable("warehouse.py", base=base, target_name="warehouse.exe")]
)
# System Magazynowy

Prosta aplikacja do zarządzania magazynem napisana w Pythonie.

## Funkcje

- Zarządzanie produktami (dodawanie, edycja, usuwanie)
- Śledzenie stanów magazynowych
- Obsługa zdjęć produktów
- Generowanie kodów produktów
- Obliczanie wartości magazynu

## Instalacja

1. Pobierz najnowszą wersję z zakładki "Releases"
2. Rozpakuj archiwum ZIP
3. Uruchom plik `warehouse.exe`

## Wymagania systemowe

- Windows 7/8/10/11
- Min. 4GB RAM
- 100MB wolnego miejsca na dysku

## Wsparcie

W przypadku problemów, utwórz nowy Issue w tym repozytorium.