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
  ```csharp
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
  ```csharp
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
  
  ### _**Exercitii:**_
  #### _*1. Implementati operatiile de Index si Details pentru entitatea considerata in cadrul proiectului propriu utilizand paradigma MVC.*_
  ##### 1.1 Proiect web de tip mvc - https://github.com/bmarian98/java_mvc

##### Referinte catre pachete "ro.ppaw.dao" = NivelAccesDate, "ro.ppaw.beans" = Librarie Model.
	
Referinta obiectului petDao din clasa PetDao.jva pachetul "ro.ppaw.dao" este trimis prin Dependancy Injection	
	
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
    xmlns:p="http://www.springframework.org/schema/p"  
    xmlns:context="http://www.springframework.org/schema/context"  
    xsi:schemaLocation="http://www.springframework.org/schema/beans  
http://www.springframework.org/schema/beans/spring-beans-3.0.xsd  
http://www.springframework.org/schema/context  
http://www.springframework.org/schema/context/spring-context-3.0.xsd">  

<context:component-scan base-package="ro.ppaw.controllers"></context:component-scan>

<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
	<property name="prefix" value="/WEB-INF/jsp/"></property>
	<property name="suffix" value=".jsp"></property>
</bean>

<bean id="ds" class="org.springframework.jdbc.datasource.DriverManagerDataSource">  
	<property name="driverClassName" value="com.mysql.jdbc.Driver"></property>  
	<property name="url" value="jdbc:mysql://localhost:3306/mvc"></property>  
	<property name="username" value="root"></property>  
	<property name="password" value="new123"></property>  
</bean>  

<bean id="jt" class="org.springframework.jdbc.core.JdbcTemplate">
<property name="dataSource" ref="ds"></property>
</bean>


<bean id="petDao" class="ro.ppaw.dao.PetDao">
	<property name="template" ref="jt"></property>
</bean>	
```	
	
  Modelul Shelter.java
  ```java
public class Shelter {

	@Override
	public String toString() {
		return "Shelter [id=" + id + ", name=" + name + ", address=" + address + "]";
	}

	private Integer id;
	private String name;
	private String address;

	public Shelter() {
	}

	public Shelter(Integer id, String name, String address) {
		super();
		this.id = id;
		this.name = name;
		this.address = address;
	}

	public void setId(Integer id) {
		this.id = id;
	}

	public Integer getId() {
		return id;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getName() {
		return name;
	}

	public void setAddress(String address) {
		this.address = address;
	}

	public String getAddress() {
		return address;
	}
}
  ```
  ##### 1.2 Controller actiunile: Index si Details din Controller
  ShelterController.java
  ```java
  // ...
  // index
 @RequestMapping("/list_shelters")
	public String list(Model m) {
		List<Shelter> list = shelterDao.getShelters();
		m.addAttribute("list", list);
		return "list_shelters";
	}
  
  //details
  @RequestMapping(value = "/shelter_details/{id}")
	public String datails(@PathVariable Integer id, Model m) {
		Shelter shelter = shelterDao.getShelter(id);
		m.addAttribute("command", shelter);
		return "shelter_details";
	}
  ```
  
  ##### 1.3 Adaugare referinta modele si acces date
  spring-servlet.xml
  ```xml
  <beans>
    <bean id="ds" class="org.springframework.jdbc.datasource.DriverManagerDataSource">  
      <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>  
      <property name="url" value="jdbc:mysql://localhost:3306/mvc"></property>  
      <property name="username" value="root"></property>  
      <property name="password" value="new123"></property>  
    </bean>  

    <bean id="jt" class="org.springframework.jdbc.core.JdbcTemplate">
     <property name="dataSource" ref="ds"></property>
    </bean>

    <bean id="shelterDao" class="com.javatpoint.dao.ShelterDao">
      <property name="template" ref="jt"></property>
    </bean>
  </beans>
  ```
  
  ##### 1.4 View-uri
  a. list_shelters.jsp
  ```jsp
<%@ taglib uri="http://www.springframework.org/tags/form" prefix="form"%>  
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>  

<h1>Shelters List</h1>
<table border="2" width="70%" cellpadding="2">
	<tr><th>Name</th><th>Details</th><th>Edit</th></tr>
	<c:forEach var="shelter" items="${list}"> 
		<tr>
			<td>${shelter.name}</td>
			<td><a href="shelter_details/${shelter.id}">Details</a></td>
			<td><a href="edit_shelter/${shelter.id}">Edit</a></td>
		</tr>
	</c:forEach>
</table>
<br/>
<a href="shelterform">Add New Shelter</a>
  ```
  b. shelter_detail.jsp
  
  ```jsp
<%@ taglib uri="http://www.springframework.org/tags/form" prefix="form"%>  
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>  

<h1>Shelter details</h1>
<table  border="2" width="70%" cellpadding="2">
	<tr><th>Id</th><th>Name</th><th>Address</th></tr>
	<tr>
	    <td><c:out value="${command.id}" /></td>
	    <td><c:out value="${command.name}" /></td>
	    <td><c:out value="${command.address}" /></td>  
      </tr>  
</table>  
<a href="/SpringMVCCRUDSimple/index.jsp">HOME</a>    
  ```	    
  
  ##### 1.5 Testare
  a. List view page
  ![shelter_list1](https://user-images.githubusercontent.com/39569343/142162552-27a15172-47e1-4ddd-b650-fbfe3ffa38ab.png)
      
  b. Details view page	
  ![shelter_details](https://user-images.githubusercontent.com/39569343/142162550-d856542c-0e96-4d42-8aa3-841cdbda3347.png)
      
 #### _*2. Implementati operatiile de Create si Edit pentru entitatea considerata in cadrul proiectului propriu utilizand paradigma MVC.*_
 ##### Metodele create si save din Controler
 ShelterController.java
 ```java
@Controller
public class ShelterController {

	@Autowired
	ShelterDao shelterDao;

	@RequestMapping("/shelterform")
	public String showform(Model m) {
		m.addAttribute("command", new Shelter());
		return "shelterform";
	}

	// insereaza datele in baza de date
	@RequestMapping(value = "/save_shelter", method = RequestMethod.POST)
	public String save(@ModelAttribute("shelter") Shelter shelter) {
		System.out.println(shelter);
		shelterDao.save(shelter);
		return "redirect:/list_shelters";
	}
	
	// extrage obiectul dupa id din baza de date si permite editarea acestuia
	@RequestMapping(value = "/edit_shelter/{id}")
	public String edit(@PathVariable Integer id, Model m) {
		Shelter shelter = shelterDao.getShelter(id);
		m.addAttribute("command", shelter);

		return "edit_shelter";
	}

	// updateaza obiectul si il salveaza in baza de date
	@RequestMapping(value = "/edit_save_shelter", method = RequestMethod.POST)
	public String editsave(@ModelAttribute("shelter") Shelter shelter) {
		shelterDao.update(shelter);

		return "redirect:/list_shelters";
	}

}
 ```
##### View-uri pentru adaugare si editare
a. shelterform.jsp
```jsp
<%@ page language="java" contentType="text/html; charset=ISO-8859-1"
    pageEncoding="ISO-8859-1"%>
<%@ taglib uri="http://www.springframework.org/tags/form" prefix="form"%>    
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>    
    
<!DOCTYPE html>
<html>
<head>
<meta charset="ISO-8859-1">
<title>Add new Shelter</title>
</head>
<body>
	<h1>Add shelter</h1>
	
	<form:form method="post" action="save_shelter">
		<table>
			<tr>
				<td>Id:</td>
				<td><form:input path="id" /></td>
			</tr>
			<tr>
				<td>Name:</td>
				<td><form:input path="name" /></td>
			</tr>
			<tr>
				<td>Address:</td>
				<td><form:input path="address" /></td>
			</tr>
			<tr>
				<td></td>
				<td><input type="submit" value="Save" /></td>
			</tr>
		</table>
	</form:form>
	<a href="/SpringMVCCRUDSimple/index.jsp">HOME</a>
</body>
</html>	
```
b. edit_helter.java
```jsp
<%@ taglib uri="http://www.springframework.org/tags/form" prefix="form"%>  
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>  

<h1>Edit shelter</h1>
<form:form method="POST" action="/SpringMVCCRUDSimple/edit_save_shelter">  
<table >  
	<tr>
	<td></td>  
	 	<td><form:hidden  path="id" /></td>
	 </tr> 
	 <tr>  
		  <td>Name : </td> 
		  <td><form:input path="name"  /></td>
	 </tr>  
	 <tr>  
		  <td>Address :</td>  
		  <td><form:input path="address" /></td>
	 </tr> 
	 <tr>  
		  <td> </td>  
		  <td><input type="submit" value="Edit Save" /></td>  
	 </tr>  
</table>  
</form:form>  
```
##### Rezultate
a. Adugare
	
![add_shelter](https://user-images.githubusercontent.com/39569343/142162540-efb94a21-90b6-4e3a-8e5d-8e50a2ba50b0.png)
	
b. Editare
	
![edit_shelter](https://user-images.githubusercontent.com/39569343/142162543-c65bebba-722c-41a2-aa16-b420a896b465.png)
![list_shelter2](https://user-images.githubusercontent.com/39569343/142162547-23075bdd-d6f0-4ce0-a6b7-3f9e56bf8319.png)
  
</details>

## _**Lab 4**_
<details>
  <summary>Apasti pentru a deschide laboratorul 4!</summary>
	
### _**Exercitii:**_
	
#### *1. Implementati operatiile de Index si Create pentru o a doua entitate considerata in cadrul proiectului propriu. Aceasta entitate trebuie sa contina o cheie straina. In acest fel, modelul implementat va contine date din mai multe tabele (In cadrul exemplului de la curs am considerat MasinaModel).*
	
##### Referinte catre pachete "ro.ppaw.dao" = NivelAccesDate, "ro.ppaw.beans" = Librarie Model.
	
Referinta obiectului petDao din clasa PetDao.jva pachetul "ro.ppaw.dao" este trimis prin Dependancy Injection	
	
Referinta obiectului shelterDao din clasa ShelterDao.jva pachetul "ro.ppaw.dao" este trimis prin Dependancy Injection	
	
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
    xmlns:p="http://www.springframework.org/schema/p"  
    xmlns:context="http://www.springframework.org/schema/context"  
    xsi:schemaLocation="http://www.springframework.org/schema/beans  
http://www.springframework.org/schema/beans/spring-beans-3.0.xsd  
http://www.springframework.org/schema/context  
http://www.springframework.org/schema/context/spring-context-3.0.xsd">  

<context:component-scan base-package="ro.ppaw.controllers"></context:component-scan>

<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
	<property name="prefix" value="/WEB-INF/jsp/"></property>
	<property name="suffix" value=".jsp"></property>
</bean>

<bean id="ds" class="org.springframework.jdbc.datasource.DriverManagerDataSource">  
	<property name="driverClassName" value="com.mysql.jdbc.Driver"></property>  
	<property name="url" value="jdbc:mysql://localhost:3306/mvc"></property>  
	<property name="username" value="root"></property>  
	<property name="password" value="new123"></property>  
</bean>  

<bean id="jt" class="org.springframework.jdbc.core.JdbcTemplate">
<property name="dataSource" ref="ds"></property>
</bean>


<bean id="petDao" class="ro.ppaw.dao.PetDao">
	<property name="template" ref="jt"></property>
</bean>	
	
<bean id="shelterDao" class="ro.ppaw.dao.ShelterDao">
	<property name="template" ref="jt"></property>
</bean>
```
	
##### Modelul Pet.java
```java
public class Pet {
	private Integer id;
	private Integer shelterId;
	private String name;
	private String dateBirth;
	private Character sex;

	public Pet() {
	}

	public Pet(Integer id, Integer shelterId, String name, String dateBirth, Character sex) {
		super();
		this.id = id;
		this.shelterId = shelterId;
		this.name = name;
		this.dateBirth = dateBirth;
		this.sex = sex;
	}

	public Integer getId() {
		return id;
	}

	public void setId(Integer id) {
		this.id = id;
	}

	public Integer getShelterId() {
		return shelterId;
	}

	public void setShelterId(Integer shelterId) {
		this.shelterId = shelterId;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getDateBirth() {
		return dateBirth;
	}

	public void setDateBirth(String dateBirth) {
		this.dateBirth = dateBirth;
	}

	public Character getSex() {
		return sex;
	}

	public void setSex(Character sex) {
		this.sex = sex;
	}

	@Override
	public String toString() {
		return "Pet [shelterId=" + shelterId + ", name=" + name + ", dateBirth=" + dateBirth + ", sex=" + sex + "]";
	}

}
```
##### Controller-ul PetControler.java
```java
@Controller()
public class PetController {

	@Autowired
	PetDao petDao;

	@RequestMapping("/petform")
	public String showform(Model m) {
		m.addAttribute("command", new Pet());
		return "petform";
	}

	@RequestMapping(value = "/save_pet", method = RequestMethod.POST)
	public String save(@ModelAttribute("pet") Pet pet) {
		petDao.save(pet);
		return "redirect:/list_pets";
	}

	@RequestMapping("/list_pets")
	public String list(Model m) {
		List<Pet> list = petDao.getPets();
		m.addAttribute("list", list);
		return "list_pets";
	}

	@RequestMapping(value = "/pet_details/{id}")
	public String detilas(@PathVariable int id, Model m) {
		Pet pet = petDao.getPet(id);
		m.addAttribute("command", pet);
		return "pet_details";
	}

	@RequestMapping(value = "/edit_tmp_pet/{id}")
	public String edit_tmp_pet(@PathVariable int id, Model m) {
		Pet pet = petDao.getPet(id);
		m.addAttribute("command", pet);
		return "pet_edit_form";
	}

	@RequestMapping(value = "/edit_save_pet", method = RequestMethod.POST)
	public String editsave(@ModelAttribute("pet") Pet pet) {
		petDao.update(pet);

		return "redirect:/list_pets";
	}
}		
```
##### View-uri pentru list, details, create si edit
a. Add - petform.jsp
```jsp
	<%@ page language="java" contentType="text/html; charset=ISO-8859-1"
    pageEncoding="ISO-8859-1"%>
<%@ taglib uri="http://www.springframework.org/tags/form" prefix="form"%>    
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>    
    
<!DOCTYPE html>
<html>
<head>
<meta charset="ISO-8859-1">
<title>Add new Pet</title>
</head>
<body>
	<h1>Add pet</h1>
	
	<form:form method="post" action="save_pet">
		<table>
			<tr>
				<td>Id:</td>
				<td><form:input path="id" /></td>
			</tr>
			<tr>
				<td>Name:</td>
				<td><form:input path="name" /></td>
			</tr>
			<tr>
				<td>ShelterId:</td>
				<td><form:input path="shelterId" /></td>
			</tr>
			<tr>
				<td>DateBirth:</td>
				<td><form:input path="dateBirth" /></td>
			</tr>
			<tr>
				<td>PetSex:</td>
				<td><form:radiobutton path="sex" value="M" />Mascul</td>
				<td><form:radiobutton path="sex" value="F" />Femela</td>
			</tr>
			<tr>
				<td></td>
				<td><input type="submit" value="Save" /></td>
			</tr>
		</table>
	</form:form>
<a href="/SpringMVCCRUDSimple/index.jsp">HOME</a>
</body>
</html>
```
	
b.  List - list_pets.jsp
```jsp
    <%@ taglib uri="http://www.springframework.org/tags/form" prefix="form"%>  
    <%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>  

	<h1>Pets List</h1>
	<table  border="2" width="70%" cellpadding="2">
	<tr><th>Pet Id</th><th>Details</th><th>Edit</th></tr>
    <c:forEach var="pet" items="${list}"> 
    <tr>
    <td>${pet.name}</td>
    <td><a href="pet_details/${pet.id}">Details</a></td>
    <td><a href="edit_tmp_pet/${pet.id}">Edit</a></td>
    </tr>
    </c:forEach>
    </table>
    <br/>
    <a href="petform">Add New Pet</a>
    <a href="/SpringMVCCRUDSimple/index.jsp">HOME</a>	
```

c. Details - pet_details.java
```java
<%@ taglib uri="http://www.springframework.org/tags/form" prefix="form"%>  
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>  

<h1>Pet details</h1>
	<table border="2" width="70%" cellpadding="2">
		<tr><th>Id</th><th>Name</th><th>Shelter_id</th><th>Date birth</th><th>Sex</th></tr>
	   	<tr>
		    <td><c:out value="${command.id}" /></td>
		    <td><c:out value="${command.name}" /></td>
		    <td><c:out value="${command.shelterId}" /></td>  
		    <td><c:out value="${command.dateBirth}" /></td>
		    <td><c:out value="${command.sex}" /></td>
	      </tr>  
     </table>  
     <a href="/SpringMVCCRUDSimple/index.jsp">HOME</a>	    
```  

d. Edit - pet_edit_form.jsp
```jsp
<%@ taglib uri="http://www.springframework.org/tags/form" prefix="form"%>  
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>  

		<h1>Edit pet</h1>
       <form:form method="POST" action="/SpringMVCCRUDSimple/edit_save_pet">  
      	<table >  
      	<tr>
      	<td></td>  
        	 <td><form:hidden  path="id" /></td>
         </tr> 
         <tr>  
		  <td>Name : </td> 
		  <td><form:input path="name"  /></td>
         </tr>  
         <tr>  
		  <td>ShelterId :</td>  
		  <td><form:input path="shelterId" /></td>
         </tr> 
         <tr>  
		  <td>DateBirth :</td>  
		  <td><form:input path="dateBirth" /></td>
         </tr> 
         <tr>  
          <td>PetSex:</td>
		 <td><form:radiobutton path="sex" value="M" />Mascul</td>
		<td><form:radiobutton path="sex" value="F" />Femela</td>
         </tr> 
         
         <tr>  
          <td> </td>  
          <td><input type="submit" value="Edit Save" /></td>  
         </tr>  
        </table>  
       </form:form>  
       <a href="/SpringMVCCRUDSimple/index.jsp">HOME</a>
```
	
##### Rezultate
a. Add view
	
![Screenshot 2021-11-17 112708](https://user-images.githubusercontent.com/39569343/142173832-ba132080-a7ed-47be-a29e-afce6501c98d.png)

b. List view
	
![pet_list_add](https://user-images.githubusercontent.com/39569343/142173008-977e1204-65f2-4e3b-817a-1ce3334cf7a3.png)

c. Edit view
	
![edit_pet](https://user-images.githubusercontent.com/39569343/142173001-f767c640-1cb5-4730-821f-6db19c78efbf.png)

d. List view

![list_edit_pet](https://user-images.githubusercontent.com/39569343/142173005-d7533a63-43cf-46d0-a7a5-af1542ee9420.png)
	
</details>

## _**Lab 5**_
<details>
  <summary>Apasti pentru a deschide laboratorul 5!</summary>
	
### _**Exercitii:**_
	
#### *1. Implementați operațiile de Get si Get(int id) pentru entitatea considerata in cadrul proiectului propriu utilizând paradigma API.*
	
Proiect WEB API
	
https://github.com/bmarian98/pp_aw
	
Crearea tabelelor din model

Pet.java
```java
@Entity
public class Pet implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(nullable = false, updatable = false)
    private Long id;
    private String name;
    private String species;
    private String dateBirth;
    private Character sex;
    private String imageUrl;

    public Pet() {}

    public Pet(Long id, String name, String species, String dateBirth, String imageUrl, Character sex) {
        this.id = id;
        this.name = name;
        this.species = species;
        this.dateBirth = dateBirth;
        this.imageUrl = imageUrl;
        this.sex = sex;
    }

    public String getImageUrl() {
        return imageUrl;
    }

    public void setImageUrl(String imageUrl) {
        this.imageUrl = imageUrl;
    }
 // ...
}
```
	
Shelter.java
```java
@Entity
public class Shelter implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(nullable = false, updatable = false)
    private Long id;

    @OneToMany(
          cascade = CascadeType.ALL,
          orphanRemoval = true
    )
    private List<Pet> pets = new ArrayList<>();
    private String name;
    private String address;

    public Shelter() {}

    public Shelter(Long id, String name, String address) {
        this.id = id;
        this.name = name;
        this.address = address;
    }
}
```

PetRepository.java - Repository
```java
@Repository
public interface PetRepository extends JpaRepository<Pet, Long> {
    void deletePetById(Long id);

    Optional<Pet> findPetById(Long id);
}
```
	
PetService.java - Service	
```java
@Service
@Transactional
public class PetService {

    private final PetRepository petRepo;

    @Autowired
    public PetService(PetRepository petRepo){
        this.petRepo = petRepo;
    }

    public Pet addPet(Pet pet){
        return petRepo.save(pet);
    }

    public List<Pet> findAllPets(){
        return petRepo.findAll();
    }

    public Pet updatePet(Pet pet){
        return petRepo.save(pet);
    }

    public void deletePet(Long id){
        petRepo.deletePetById(id);
    }

    public Pet findPetById(Long id){
        return petRepo.findPetById(id).orElseThrow(() -> new UserNotFoundException("Pet with id " + id + " not found"));
    }

}	
```
	
PetResource.java - RestController
```java
@RestController
@RequestMapping("/pet")
public class PetResource {
    private final PetService petService;

    public PetResource(PetService petService){
        this.petService = petService;
    }

    // GET pentru toate elementele din table
    @GetMapping("/all")
    public ResponseEntity<List<Pet>> getAllPets(){
        List<Pet> pets = petService.findAllPets();
        return new ResponseEntity<>(pets, HttpStatus.OK);
    }

    // GET cu ID
    @GetMapping("/find/{id}")
    public ResponseEntity<Pet> getPetById(@PathVariable("id") Long id){
        Pet pet = petService.findPetById(id);
        return new ResponseEntity<>(pet, HttpStatus.OK);
    }

}
```
	
#### Testare cu Postman

Get pentru toate elementele
![find_all](https://user-images.githubusercontent.com/39569343/143069933-c2e950bd-a838-4471-9f2e-5cafb0fcc35d.png)
	
Get pentru un element cu id specific
![find_by_id](https://user-images.githubusercontent.com/39569343/143069943-c0de19f9-30a8-494b-aaa1-ceed68ab3df8.png)
	
#### *2. Implementați operațiile de Post si Put pentru entitatea considerata in cadrul proiectului propriu utilizând paradigma API.*	
	
PetResource.java - RestController
```java
@RestController
@RequestMapping("/pet")
public class PetResource {
    private final PetService petService;

    public PetResource(PetService petService){
        this.petService = petService;
    }
	
    // ...

    @PostMapping("/add")
    public ResponseEntity<Pet> addPet(@RequestBody Pet pet){
        Pet newPet = petService.addPet(pet);
        return new ResponseEntity<>(newPet, HttpStatus.OK);
    }

    @PutMapping("/edit")
    public ResponseEntity<Pet> editPet(@RequestBody Pet pet){
        Pet editPet = petService.updatePet(pet);
        return new ResponseEntity<>(editPet, HttpStatus.OK);
    }
}
```

#### Testare POST si PUT 
	
![post_pet](https://user-images.githubusercontent.com/39569343/143071004-f70208f5-25ef-4aa3-933d-4216059952e1.png)

![edit_put](https://user-images.githubusercontent.com/39569343/143071039-2bdaffe8-dbb0-4ed6-9c15-632a5d2d817f.png)
	
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
