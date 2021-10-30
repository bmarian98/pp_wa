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
