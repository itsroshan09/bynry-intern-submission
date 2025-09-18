# Bynry - Backend Engineering Intern Case Study

This repository contains my complete solution for the Bynry backend case study.

---

## 📂 Repository Structure

-   **/PART_1_CODE_REVIEW.md**: Contains the detailed analysis of the provided code snippet, including identified issues, their impact, and the corrected version.
-   **/PART_2_DATABASE_DESIGN.md**: Outlines the proposed database schema, questions for the product team, and key design justifications.
-   **/project/**: A Django project containing the implementation for the low-stock alerts API endpoint from Part 3.

---

## 🛠️ Part 3: Running the API Implementation

### Prerequisites

-   Python 3.8+
-   pip

### Setup Instructions

1.  **Clone the repository:**
    ```bash
    git clone [https://github.com/your-username/bynry-backend-casestudy.git](https://github.com/your-username/bynry-backend-casestudy.git)
    cd bynry-backend-casestudy
    ```

2.  **Create and activate a virtual environment:**
    ```bash
    python -m venv venv
    source venv/bin/activate  # On Windows, use `venv\Scripts\activate`
    ```

3.  **Install the required packages:**
    ```bash
    pip install -r requirements.txt
    ```

4.  **Run the application (from inside the `project` directory):**
    ```bash
    cd project
    python manage.py migrate
    python manage.py runserver
    ```
The API will be available at `http://127.0.0.1:8000/`.

---

## 🤔 Assumptions Made

-   **Recent Sales:** "Recent sales activity" is defined as at least one sale occurring in the last 30 days.
-   **Supplier Info:** For products with multiple suppliers, the alert displays the first supplier associated with the product.
-   _(Add any other assumptions you made here)_
