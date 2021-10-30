# PPAW

## _**Lab 1**_
<details unu>
  <summary>Apasati pentru a extinde laboratorul 1!</summary>

### **Exercitii:**
#### _*1. Adaugati un nou camp clasei Masina din cadrul proiectului demo si modificati toate nivelurile aplicatiei WinForms sa reflecte aceasta schimbare.*_

_*Campul adaugat este **Culoare** de tip string.*_
  
**a. Masina.cs**
```csharp
namespace LibrarieModele{
    public class Masina {
      // ...
      public string Culoare {get; set;}
  
      public Masina(DateTime dataFabricatie, int idCompanie, string model, decimal pret, string culoare, int idMasina = 0)
      {
        // ...
        Culoare = culoare;
      }
  
      public Masina (DataRow linieBD)
      {
          // ...
          Culoare = linieBD["culoare"].ToString();
      }
    } // class Masina
  } // namespace Librarie Modele
```

**b. AdministrareMasini.cs**
```csharp  
  namespace NivelAccesDate_SQLServer
{
    public class AdministrareMasini: IStocareMasini
    {
       public bool AddMasina(Masina m)
        {
            return SqlDBHelper.ExecuteNonQuery(
                "insert into dbo.Masini VALUES (@DataFabricatie, @IdCompanie, @Model, @Pret, @Culoare)", CommandType.Text,
                // ...
                new SqlParameter("@Culoare", m.Culoare)
            );
        }

        public bool UpdateMasina(Masina m)
        {
            return SqlDBHelper.ExecuteNonQuery(
                "UPDATE dbo.Masini set dataFabricatie = @DataFabricatie, idCompanie = @IdCompanie, model =@Model, pret =@Pret, culoare = @Culoare where idMasina=@IdMasina", CommandType.Text,
                // ...
                new SqlParameter("@Culoare", m.Culoare)
            );
        }
    } // class AdministrareMasini
} // namespace NivelAccesDate_SQLServer  
```
  
**c. FormaAdaugare.cs**
```csharp
namespace InterfataUtilizator
{
    public partial class FormaAdaugare : MetroForm
    {
      // ...
       private void dataGridMasini_CellContentClick(object sender, DataGridViewCellEventArgs e)
        {
            this.Width = 1000;
            int currentRowIndex = dataGridMasini.CurrentCell.RowIndex;
            string idMasina = dataGridMasini[PRIMA_COLOANA, currentRowIndex].Value.ToString();

            try
            {
                Masina m = stocareMasini.GetMasina(Int32.Parse(idMasina));

                //incarcarea datelor in controalele de pe forma
                if (m != null)
                {
                    // ...
                    txtCuloare.Text = m.Culoare; // adaugarea campului Culoare
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message.ToString());
            }
            pnlEditare.Visible = true;
        }

        private void btnActualizeaza_Click(object sender, EventArgs e)
        {
            try
            {
                var masina = new Masina(
                    dtDataFabricatie.Value,
                    ((ComboItem)cmbCompanii.SelectedItem).Value,
                    txtModel.Text,
                    decimal.Parse(txtPret.Text),
                    txtCuloare.Text, // adaugarea metroBox-ului Culoare
                    Int32.Parse(lblIdMasina.Text));

                var rezultat = stocareMasini.UpdateMasina(masina);
                if (rezultat == SUCCES)
                {
                    MessageBox.Show("Masina actualizata");
                    AfiseazaCatalog();
                }
                else
                {
                    MessageBox.Show("Eroare la actualizare masina");
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show("Exceptie" + ex.Message);
            }
        }

      // ...      
    } // class FormaAdaugare
} // namespace InterfataUtilizator
```
  
**d. FormaAfisare**
  
```csharp
namespace InterfataUtilizator
{
    public partial class FormaAfisare : MetroForm
    {
       // ...
    
        private void AfiseazaCatalog()
        {
            try
            {
                var masini = stocareMasini.GetMasini();
                if (masini != null && masini.Any())
                { // adugare campului "Culoare" in Select
                    dataGridMasini.DataSource = masini.Select(m=> new { m.IdMasina, m.Model, m.Companie.Nume, m.DataFabricatie, m.Pret, m.Culoare }).ToList() ;

                    dataGridMasini.Columns[0].Visible = false;
                    dataGridMasini.Columns[2].HeaderText = "Companie";
                    dataGridMasini.Columns[3].HeaderText = "Data fabricatie";
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message.ToString());
            }
        }
    
    } // class FormaAfisare
 } // namespace InterfataUtilizator
  
```

### *Rezultate:*
  
**1. Afisare si modificare date masina**
  
  ![afisare_modoficare_masini](https://user-images.githubusercontent.com/39569343/139529075-649f81d6-4473-457f-a57a-52e077e768f7.png)
  
**2. Adaugare masina**  
  
  ![adaugare_masina](https://user-images.githubusercontent.com/39569343/139529176-c3b16af7-296c-40ec-9762-44aca0298b84.png)
</details>

## _**Lab 2**_
<details>
  <summary>Apasati pentru a deschide laboratorul 2!</summary>
  
  ### _**Exercitii:**_
  #### *2. Adaugati o noua forma care sa afiseze toate companiile introduse.*
  
  **a. ListaCompanii.aspx**
  ```csharp
  <%@ Page Title="" Language="C#" MasterPageFile="~/Site.Master" AutoEventWireup="true" CodeBehind="ListaCompanii.aspx.cs" Inherits="InterfataUtilizator_WebForms.ListaCompanii" %>
<asp:Content ID="Content1" ContentPlaceHolderID="MainContent" runat="server">
    <section>
        <div>
            <hgroup>
                <h2><%: Page.Title %></h2>
            </hgroup>

            <asp:ListView ID="productList" runat="server" 
                DataKeyNames="IdCompanie" GroupItemCount="4"
                ItemType="LibrarieModele.Companie" SelectMethod="GetCompanii">
                <EmptyDataTemplate>
                    <table >
                        <tr>
                            <td>Nu a fost gasita nici o companie.</td>
                        </tr>
                    </table>
                </EmptyDataTemplate>
                <EmptyItemTemplate>
                    <td/>
                </EmptyItemTemplate>
                <GroupTemplate>
                    <tr id="itemPlaceholderContainer" runat="server">
                        <td id="itemPlaceholder" runat="server"></td>
                    </tr>
                </GroupTemplate>
                <ItemTemplate>
                    <td runat="server">
                        <table>
                            <tr>
                                <td>
                                    <a href="ProductDetails.aspx?productID=<%#:Item.IdCompanie%>">
                                        <img src="./Resources/Images/<%#:Item.Nume%>.jpg"
                                            width="100" height="75" style="border: solid" /></a>
                                </td>
                            </tr>
                            <tr>
                                <td>
                                    <a href="ProductDetails.aspx?productID=<%#:Item.IdCompanie%>">
                                        <span>
                                            <%#:Item.Nume%>
                                        </span>
                                    </a>
                                    <br />
                                    <span>
                                        <b>Adresa: </b><%#:Item.Adresa%>
                                    </span>
                                    <br />
                                </td>
                            </tr>
                            <tr>
                                <td>
                                    <a href="ProductDetails.aspx?productID=<%#:Item.IdCompanie%>">
                                    </a>
                                    <br />
                                    <span>
                                        <b>Email: </b><%#:Item.Email%>
                                    </span>
                                    <br />
                                </td>
                            </tr>
                            <tr>
                                <td>
                                    <a href="ProductDetails.aspx?productID=<%#:Item.IdCompanie%>">
                                    </a>
                                    <br />
                                    <span>
                                        <b>Telefon: </b><%#:Item.Telefon%>
                                    </span>
                                    <br />
                                </td>
                            </tr>
                            <tr>
                                <td>&nbsp;</td>
                            </tr>
                        </table>
                        </p>
                    </td>
                </ItemTemplate>
                <LayoutTemplate>
                    <table style="width:100%;">
                        <tbody>
                            <tr>
                                <td>
                                    <table id="groupPlaceholderContainer" runat="server" style="width:100%">
                                        <tr id="groupPlaceholder"></tr>
                                    </table>
                                </td>
                            </tr>
                            <tr>
                                <td></td>
                            </tr>
                            <tr></tr>
                        </tbody>
                    </table>
                </LayoutTemplate>
            </asp:ListView>
        </div>
    </section>
</asp:Content>
 ```
**b. Lista Companii.aspx.cs**
  ```cshar
  namespace InterfataUtilizator_WebForms
{
    public partial class ListaCompanii : System.Web.UI.Page
    {
        //initializare obiecte utilizate pentru salvarea datelor in baza de date (sau alte medii de stocare...daca exista implementare corespunzatoare)
        IStocareCompanii stocareCompanii = (IStocareCompanii)new StocareFactory().GetTipStocare(typeof(Companie));
        IStocareMasini stocareMasini = (IStocareMasini)new StocareFactory().GetTipStocare(typeof(Masina));

        protected void Page_Load(object sender, EventArgs e)
        {

        }
        public IQueryable<Companie> GetCompanii()
        {
            return stocareCompanii.GetCompanii().AsQueryable();
        }
    } // parial class ListaCompanii
} // namespace InterfataUtilizator_WebForms
  ```
**c. ListaCompanii.aspx.designer.cs**
  ```cshar
  namespace InterfataUtilizator_WebForms
{
    public partial class ListaCompanii
    {
        protected global::System.Web.UI.WebControls.ListView productList;
    } 
}
  ```
  
  **d. Site.Master**
  ```html
  <div class="navbar-collapse collapse">
      <ul class="nav navbar-nav">
          <li><a runat="server" href="~/">Home</a></li>
          <li><a runat="server" href="~/ListaMasini">Lista masini</a></li>
          <li><a runat="server" href="~/ListaCompanii">Lista companii</a></li> <!--Adaugare tab lista companii-->
          <li><a runat="server" href="~/AdaugareCompanie">AdaugareCompanie</a></li>
      </ul>
  </div>
  ```
  
  ### *Rezultate:*
  
  ![SiteMaster](https://user-images.githubusercontent.com/39569343/139533821-cb855d9b-a85b-41cc-a8b6-6a7337ee6749.png)
  
</details>

## _**Lab 3**_
<details>
  <summary>Apasti pentru a deschide laboratorul 3!</summary>
</details>

## _**Lab 4**_
<details>
  <summary>Apasti pentru a deschide laboratorul 4!</summary>
</details>

## _**Lab 5**_
<details>
  <summary>Apasti pentru a deschide laboratorul 5!</summary>
</details>

## _**Lab 6**_
<details>
  <summary>Apasti pentru a deschide laboratorul 6!</summary>
</details>

## _**Lab 7**_
<details>
  <summary>Apasti pentru a deschide laboratorul 7!</summary>
</details>

## _**Lab 8**_
<details>
  <summary>Apasti pentru a deschide laboratorul 8!</summary>
</details>

## _**Lab 9**_
<details>
  <summary>Apasti pentru a deschide laboratorul 9!</summary>
</details>

## _**Lab 10**_
<details>
  <summary>Apasti pentru a deschide laboratorul 10</summary>
</details>

## _**Lab 11**_
<details>
  <summary>Apasti pentru a deschide laboratorul 11!</summary>
</details>
