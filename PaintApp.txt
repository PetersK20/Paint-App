import java.awt.*;
import java.awt.datatransfer.*;
import java.awt.event.*;
import java.awt.geom.*;
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;
import java.sql.*;
import java.sql.Statement;
import java.text.DecimalFormat;
import java.util.ArrayList;
import java.util.HashMap;
import javax.imageio.ImageIO;
import javax.swing.*;
import javax.swing.event.*;
import javax.swing.filechooser.FileFilter;
import javax.swing.filechooser.FileNameExtensionFilter;
import javax.swing.plaf.ColorUIResource;


@SuppressWarnings("serial")
public class PaintApp extends JFrame {
	private buttonListen actionListen = new buttonListen();
	private static Color fillColor = Color.BLACK; private Color strokeColor = Color.BLACK;
	private static boolean popupVisible=false,lineSelected = true, rectangleSelected = false, ellipseSelected = false, brushSelected = false,moveSelected = false,eraserOn = false, selected = false;
	private static PaintApp paintApp;
	private image imagePanel;
	private static JPanel colorPanel,selectedColorHolder;
	private static Box northBox, shapeBox, toolbarBox;
	private static JButton select, eraser, delete, fColor, sColor, brush, move,addHorizontal, addVertical,apply,line, rectangle, ellipse;
	private static JSlider thickness, transparent;
	private boolean allReadySaved = true;
	private static Font font = new Font("Serif", Font.BOLD, 20);
	private static DecimalFormat sliderFormat;
	private Graphics2D graphics;
	private static JMenuBar menuHolder;
	private static JMenu editMenu, fileMenu;
	private static JMenuItem cut,copy,paste,redo,undo,changeColor,saveAll,openFile,gradient, changeTransparency, selectAll, deleteFromEdit, addNewCanvas, saveAs, save, addToToolBar;
	private menuListen menuListener = new menuListen();
	private static Dimension colorSize = new Dimension(30, 30);
	private mouseListen mouseListener = new mouseListen();
	private static JTextField red, green, blue;
	private static JLabel redL = new JLabel("Red:     "), greenL = new JLabel("Green: "), blueL = new JLabel("Blue:    ");
	private static JFrame colorFrame, ToolBarAdd;
	private static HashMap<String, JButton> buttonMap = new HashMap<String, JButton>();
	private static ArrayList<JCheckBox> checkBoxList = new ArrayList<JCheckBox>();
	private static ArrayList<JButton> toolboxList = new ArrayList<JButton>(); private ArrayList<JButton> colorButtonHolder=new ArrayList<JButton>();
	private static JTabbedPane multipleCanvas;
	private static Connection con;
	private static ResultSet resultSet;
	private static ArrayList<PaintApp> paintAppList;
	private static ArrayList<File> filenameList;
	private GridBagConstraints constraints;
	private JButton setFirst, setSecond;
	private JCheckBox gradientInMiddle;
	private JTextField xPos;
	private GradientPaint paint;
	private Box holdSpace;
	private JPanel firstColor, secondColor;
	private JButton createGradient;
	private static ArrayList<JPanel> deletePanels;
	private boolean isHorizontal;
	private int mementoIndex=0,mementoNumber=0;;
	private CareTaker careTaker=new CareTaker();
	private Originator originator=new Originator();
	private ArrayList<paintCompoenent>list;
	private ArrayList<Memento> undoIndexes=new ArrayList<Memento>();
	public static void main(String[] args) {
		paintAppList=new ArrayList<PaintApp>();
		paintApp = new PaintApp(true,"");
		paintAppList.add(paintApp);
		filenameList=new ArrayList<File>();
		String home=System.getProperty("user.home");
		String number=(filenameList.isEmpty())?"":Integer.toString(filenameList.size());
		File file=new File(home+"\\Downloads\\canvas"+number+".png");
		int number2;
		if(!number.equals("")){number2=Integer.parseInt(number);}
		else{number2=0;}
		while(file.exists()){
			number2++;
		file=new File(home+"\\Downloads\\canvas"+number2+".png");
		}
		try {
			file.createNewFile();
		} catch (IOException e1) {
			// TODO Auto-generated catch block
			e1.printStackTrace();
		}
		filenameList.add(file);
	}// End of main
	public PaintApp(boolean firstTime,String fileName) {
if(firstTime){
		this.setSize(1650, 870);
		this.setBackground(new Color(102, 178, 255));
		this.setLocationRelativeTo(null);
		this.setDefaultCloseOperation(JFrame.DISPOSE_ON_CLOSE);
		this.setTitle("pictureDoc1");
		this.createConnection();
		this.createEditMenu();
		this.createFileMenu();
		this.createLayout();
		this.createToolbar(true);
		this.setVisible(true);
}else{
	ImageIcon icon = new ImageIcon("C:/Users/Kyle Peters/Documents/picturesForPaintAppProject/delete.png");
	Image image = icon.getImage();
	image = image.getScaledInstance(10, 10, Image.SCALE_SMOOTH);
	icon = new ImageIcon(image);
	imagePanel=new image();
	multipleCanvas.addTab(fileName, icon,imagePanel);
	multipleCanvas.setTabComponentAt(multipleCanvas.getTabCount()-1, createTabComponents(new Color(102, 178, 255),fileName,"delete.png"));
}
	}// End of PaintApp Constructor

	private void createConnection() {
		
			try {
				String forName="com.mysql.jdbc.Driver";
				Class.forName(forName);
				String database="jdbc:mysql://localhost/paintapp";
				String user="root";
				String password="Coke6008337";
				con=DriverManager.getConnection(database,user,password);
				Statement state=con.createStatement(ResultSet.TYPE_SCROLL_SENSITIVE, ResultSet.CONCUR_UPDATABLE);
				resultSet=state.executeQuery("Select * from info");
				toolbarBox = Box.createHorizontalBox();
			}catch (ClassNotFoundException e) {
				e.printStackTrace();
			} catch (SQLException e) {
				e.printStackTrace();
			}	
		
	}//End of createConnection

	private void createLayout() {
		this.setLayout(new GridBagLayout());
		shapeBox = createShapeBox();
		northBox = createNorthPanel();
		imagePanel = new image();
		menuHolder = createTopPanel();
		sliderFormat = new DecimalFormat("#.##");
		createJTabbedPane();
		GridBagConstraints constraints = new GridBagConstraints();
		addWithGridBag(1, 1, 1, 1, 0, 0, GridBagConstraints.HORIZONTAL, GridBagConstraints.NORTHWEST, constraints, this,
				menuHolder);
		addWithGridBag(1, 2, 1, 1, 0, 0, GridBagConstraints.BOTH, GridBagConstraints.NORTHWEST, constraints, this,
				toolbarBox);
		constraints.insets = new Insets(0, 0, 5, 0);
		addWithGridBag(1, 3, 1, 1, 0, 0, GridBagConstraints.BOTH, GridBagConstraints.NORTHWEST, constraints, this,
				northBox);

		addWithGridBag(1, 4, 1, 1, 100, 100, GridBagConstraints.BOTH, GridBagConstraints.CENTER, constraints, this,
				multipleCanvas);

	}// End of createLayout
private void createJTabbedPane(){
	UIManager.put("TabbedPane.selected", new ColorUIResource(Color.WHITE));
	UIManager.put("TabbedPane.contentAreaColor", Color.WHITE);
	UIManager.put("TabbedPane.tabAreaBackground", new Color(102, 178, 255));
	UIManager.put("TabbedPane.unselectedBackground", new Color(102, 178, 255));
	
	ImageIcon icon = new ImageIcon("C:/Users/Kyle Peters/Documents/picturesForPaintAppProject/delete.png");
	Image image = icon.getImage();
	image = image.getScaledInstance(10, 10, Image.SCALE_SMOOTH);
	icon = new ImageIcon(image);
	
	deletePanels=new ArrayList<JPanel>();
	multipleCanvas=new JTabbedPane();
	multipleCanvas.addMouseListener(mouseListener);
	multipleCanvas.addTab("Canvas One",icon,imagePanel,"close");
	addListenerToTabs();
	multipleCanvas.setTabComponentAt(0, createTabComponents(Color.WHITE,"Canvas0","delete.png"));
}//End of createJTabbedPane
private void addListenerToTabs(){
	multipleCanvas.addMouseListener(new MouseAdapter(){
		public void mouseClicked(MouseEvent e) {
			if(e.getClickCount()>=2&& multipleCanvas.getBoundsAt(multipleCanvas.getSelectedIndex()).intersects(e.getX(),e.getY(),1,1)){
				int index=multipleCanvas.getSelectedIndex();
				JTextField field=new JTextField(10);
				multipleCanvas.setTabComponentAt(index, field);
				field.requestFocus();
				field.addFocusListener(new FocusAdapter(){
					public void focusLost(FocusEvent e) {
						JLabel label=(JLabel) deletePanels.get(index).getComponent(0);
						label.setText(field.getText());
					multipleCanvas.setTabComponentAt(index, deletePanels.get(index));
						}});
				field.addKeyListener(new KeyAdapter(){
					public void keyTyped(KeyEvent e) {
					if(e.getKeyChar()=='\n'){
						JLabel label=(JLabel) deletePanels.get(index).getComponent(0);
						label.setText(field.getText());
					multipleCanvas.setTabComponentAt(index, deletePanels.get(index));
					}}});
		}}});
	multipleCanvas.addChangeListener(new ChangeListener(){		
		public void stateChanged(ChangeEvent e) {
			for(JPanel panel:deletePanels){
				if(deletePanels.indexOf(panel)==multipleCanvas.getSelectedIndex()){
					panel.setBackground(Color.WHITE); panel.getComponent(0).setBackground(Color.WHITE);panel.getComponent(1).setBackground(Color.WHITE);
				}else{
					panel.setBackground(new Color(102, 178, 255)); panel.getComponent(0).setBackground(new Color(102, 178, 255));panel.getComponent(1).setBackground(new Color(102, 178, 255));
		}}}});
}//End if addListenersToTabs
private JPanel createTabComponents(Color color,String label,String iconName){
	ImageIcon icon = new ImageIcon("C:/Users/Kyle Peters/Documents/picturesForPaintAppProject/" + iconName);
	Image image = icon.getImage();
	image = image.getScaledInstance(10, 10, Image.SCALE_SMOOTH);
	icon = new ImageIcon(image);
	
	JPanel panel=new JPanel();
	panel.setOpaque(true);
	panel.setBackground(color);
	JLabel label2=new JLabel(label);
	label2.setName(label);
	panel.add(label2);

	JButton button=new JButton();
	button.setIcon(icon);
	button.setBackground(color);
	button.setContentAreaFilled(false);
	button.setFocusable(false);
	button.setBorder(BorderFactory.createEtchedBorder());
	button.setBorderPainted(false);
	panel.add(button);
	button.addActionListener(new ActionListener(){
		public void actionPerformed(ActionEvent e) {
			for(JPanel panel:deletePanels){
				if(e.getSource()==panel.getComponent(1)){
					int index=multipleCanvas.indexOfTab(panel.getComponent(0).getName());
						if(index==multipleCanvas.getSelectedIndex()){
								if(multipleCanvas.getTabCount()>1){
										multipleCanvas.removeTabAt(index);
										deletePanels.remove(panel);
									try{
										deletePanels.get(multipleCanvas.getSelectedIndex()).setBackground(Color.WHITE); deletePanels.get(multipleCanvas.getSelectedIndex()).getComponent(0).setBackground(Color.WHITE);deletePanels.get(multipleCanvas.getSelectedIndex()).getComponent(1).setBackground(Color.WHITE);
									}catch(IndexOutOfBoundsException exception){}
									break;
								}else{
									JOptionPane.showMessageDialog(null, "You Can't delete the last tab","Information",JOptionPane.INFORMATION_MESSAGE);
								}
						}else{
							multipleCanvas.setSelectedIndex(index);
							break;
				}}}}});//End of ActionListener
	deletePanels.add(panel);
	return panel;
}//End ofcreateTabComponenets
	private void reCreateLayout() {
		GridBagConstraints constraints = new GridBagConstraints();
		addWithGridBag(1, 1, 1, 1, 0, 0, GridBagConstraints.HORIZONTAL, GridBagConstraints.NORTHWEST, constraints, this,
				menuHolder);
		addWithGridBag(1, 2, 1, 1, 0, 0, GridBagConstraints.HORIZONTAL, GridBagConstraints.NORTHWEST, constraints, this,
				toolbarBox);
		constraints.insets = new Insets(0, 0, 5, 0);
		addWithGridBag(1, 3, 1, 1, 0, 0, GridBagConstraints.BOTH, GridBagConstraints.NORTHWEST, constraints, this,
				northBox);

		addWithGridBag(1, 4, 1, 3, 100, 100, GridBagConstraints.BOTH, GridBagConstraints.CENTER, constraints, this,
				multipleCanvas);
		this.revalidate();
	}//End of reCreateLayout

	private void createToolbar(boolean firstTime) {
		toolbarBox.removeAll();
		if(firstTime){
			for (JCheckBox checkBox : checkBoxList) {
				if (checkBox.isSelected()) {
					toolboxList.add(buttonMap.get(checkBox.getText()));
				}
			}
		}
		for (JButton button : toolboxList) {
			button.setBackground(new Color(204, 229, 255));
			button.setBorderPainted(false);
			button.setFocusPainted(false);
			toolbarBox.add(button);
		}
		toolbarBox.setOpaque(true);
		toolbarBox.setBackground(new Color(204, 229, 255));
	}//End of createToolbar

	private JMenuBar createTopPanel() {
		JMenuBar menuBar=new JMenuBar();
		menuBar.setBackground(Color.WHITE);
		menuBar.add(fileMenu);
		menuBar.add(editMenu);
		menuBar.setBackground(Color.WHITE);
		fileMenu.addMenuListener(new MenuListener(){
			public void menuSelected(MenuEvent arg0) {
				paintAppList.get(multipleCanvas.getSelectedIndex()).modifyFileMenu();		
			}
			public void menuCanceled(MenuEvent e) {}
			public void menuDeselected(MenuEvent e) {}
			});
		editMenu.addMenuListener(new MenuListener(){
			public void menuSelected(MenuEvent arg0) {
				paintAppList.get(multipleCanvas.getSelectedIndex()).modifyEditMenu();		
			}
			public void menuCanceled(MenuEvent e) {}
			public void menuDeselected(MenuEvent e) {}
			});
		return menuBar;
	}// End of createTopPanel

	private Box createShapeBox() {
		JPanel panel = new JPanel();
		panel.setLayout(new GridLayout(0, 3));
		Box shapeBox = Box.createVerticalBox();

		JLabel label = new JLabel("Shapes");
		label.setHorizontalAlignment(JLabel.CENTER);
		label.setFont(font);

		line = makeButtons("line.jpg", "", 30, 30, false, panel, true, actionListen);

		rectangle = makeButtons("rectangle.jpg", "", 30, 30, false, panel, true, actionListen);

		ellipse = makeButtons("ellipse.jpg", "", 30, 30, false, panel, true, actionListen);

		JScrollPane pane = new JScrollPane(panel, JScrollPane.VERTICAL_SCROLLBAR_AS_NEEDED,
				JScrollPane.HORIZONTAL_SCROLLBAR_NEVER);

		shapeBox.add(label);
		shapeBox.add(pane);
		shapeBox.setOpaque(true);
		shapeBox.setBackground(new Color(153, 204, 255));
		return shapeBox;
	}// End of createShapeBox

	private Box createNorthPanel() {
		Box northBox = Box.createHorizontalBox();
		select = makeButtons("mouseIcon.png", "     Select", 70, 60, true, northBox, true, actionListen);
		move = makeButtons("moveCursor.png", "Move Shapes", 70, 60, true, northBox, true, actionListen);
		eraser = makeButtons("eraser.jpg", "     Eraser", 70, 60, true, northBox, true, actionListen);
		brush = makeButtons("paintBrush.jpg", "      Brush", 70, 60, true, northBox, true, actionListen);
		delete = makeButtons("delete.png", "     Delete", 70, 60, true, northBox, true, actionListen);
		fColor = makeButtons("fillColor.jpg", "  Fill Color", 70, 60, true, northBox, true, actionListen);
		sColor = makeButtons("strokeColor.jpg", "Stroke Color", 70, 60, true, northBox, true, actionListen);

		northBox.add(shapeBox);
		thickness = createJSliders("Thickness : 0.10", northBox, 10);
		transparent = createJSliders("Transparency : 1.00", northBox, 100);
		northBox.setOpaque(true);
		northBox.setBackground(new Color(153, 204, 255));
		return northBox;
	}// End of createNorthPanel

	private void createColorPanel() {
		colorButtonHolder = new ArrayList<JButton>();
		colorPanel = new JPanel();
		colorPanel.setLayout(new GridLayout(1, 13, 5, 5));
		pattern1(true, false, false);
		pattern3(true, false, false, false, true, false);
		pattern2(true, false, false);
		pattern3(false, true, false, true, false, false);
		pattern1(false, true, false);
		pattern3(false, true, false, false, false, true);
		pattern2(false, true, false);
		pattern3(false, false, true, false, true, false);
		pattern1(false, false, true);
		pattern3(false, false, true, true, false, false);
		pattern2(false, false, true);
		pattern3(true, false, false, false, false, true);

		Box box = Box.createVerticalBox();
		for (int x = 0; x < 255; x += 32) {
			createColorButtons(x, x, x, box);
			box.add(Box.createVerticalStrut(5));
		}
		createColorButtons(255, 255, 255, box);
		colorPanel.add(box);

	}// End of createColorPanel

	private void pattern1(boolean firstIncre, boolean secondIncre, boolean thirdIncre) {
		Box box = Box.createVerticalBox();
		for (int x = 51; x <= 255; x += 51) {
			if (firstIncre) {
				createColorButtons(x, 0, 0, box);
			} else if (secondIncre) {
				createColorButtons(0, x, 0, box);
			} else {
				createColorButtons(0, 0, x, box);
			}
			box.add(Box.createVerticalStrut(5));
		}
		for (int x = 51; x < 255; x += 51) {
			if (firstIncre) {
				createColorButtons(255, x, x, box);
			} else if (secondIncre) {
				createColorButtons(x, 255, x, box);
			} else {
				createColorButtons(x, x, 255, box);
			}
			box.add(Box.createVerticalStrut(5));
		}
		colorPanel.add(box);
	}// End of pattern1

	private void pattern2(boolean firstIncre, boolean secondIncre, boolean thirdIncre) {
		Box box = Box.createVerticalBox();
		for (int x = 51; x <= 255; x += 51) {
			if (firstIncre) {
				createColorButtons(x, x, 0, box);
			} else if (secondIncre) {
				createColorButtons(0, x, x, box);
			} else {
				createColorButtons(x, 0, x, box);
			}
			box.add(Box.createVerticalStrut(5));
		}
		for (int x = 51; x < 255; x += 51) {
			if (firstIncre) {
				createColorButtons(255, 255, x, box);
			} else if (secondIncre) {
				createColorButtons(x, 255, 255, box);
			} else {
				createColorButtons(255, x, 255, box);
			}
			box.add(Box.createVerticalStrut(5));
		}
		colorPanel.add(box);
	}// End of pattern2

	private void pattern3(boolean first51, boolean second51, boolean third51, boolean first25, boolean second25,
			boolean third25) {
		Box box = Box.createVerticalBox();
		int y = 25;
		int checkYIncrement = 25;
		for (int x = 51; x <= 255; x += 51) {
			if (first51 && second25) {
				createColorButtons(x, y, 0, box);
			} else if (first51 && third25) {
				createColorButtons(x, 0, y, box);
			} else if (second51 && first25) {
				createColorButtons(y, x, 0, box);
			} else if (second51 && third25) {
				createColorButtons(0, x, y, box);
			} else if (third51 && first25) {
				createColorButtons(y, 0, x, box);
			} else {
				createColorButtons(0, y, x, box);
			}
			if (checkYIncrement == 25) {
				y += 26;
				checkYIncrement = 26;
			} else {
				y += 25;
				checkYIncrement = 25;
			}
			box.add(Box.createVerticalStrut(5));
		}
		for (int x = 51; x < 255; x += 51) {
			if (first51 && second25) {
				createColorButtons(255, y, x, box);
			} else if (first51 && third25) {
				createColorButtons(255, x, y, box);
			} else if (second51 && first25) {
				createColorButtons(y, 255, x, box);
			} else if (second51 && third25) {
				createColorButtons(x, 255, y, box);
			} else if (third51 && first25) {
				createColorButtons(y, x, 255, box);
			} else {
				createColorButtons(x, y, 255, box);
			}
			if (checkYIncrement == 25) {
				y += 26;
				checkYIncrement = 26;
			} else {
				y += 25;
				checkYIncrement = 25;
			}
			box.add(Box.createVerticalStrut(5));

		}
		colorPanel.add(box);
	}// End of patter3

	private void createColorButtons(int r, int g, int b, Box box) {
		JButton button = new JButton();
		button.addMouseListener(mouseListener);
		button.setPreferredSize(colorSize);
		button.setMaximumSize(colorSize);
		button.setMinimumSize(colorSize);
		button.setBackground(new Color(r, g, b));
		box.add(button);
		colorButtonHolder.add(button);
	}// End of createColorButtons

	public static void addWithGridBag(int x, int y, int width, int height, int weightX, int weightY, int fill,
			int anchor, GridBagConstraints constraints, JFrame p, JComponent c) {

		constraints.gridx = x;
		constraints.gridy = y;
		constraints.weightx = weightX;
		constraints.weighty = weightY;

		constraints.gridwidth = width;

		constraints.gridheight = height;

		constraints.fill = fill;

		constraints.anchor = anchor;

		p.add(c, constraints);
	}// End of addWithGridBag

	private JSlider createJSliders(String lableName, Box box, int increment) {
		Box sliderBox = Box.createVerticalBox();
		JLabel lable = new JLabel(lableName);
		lable.setFont(font);
		JSlider slider = new JSlider(0, 100, increment);
		slider.setBackground(new Color(153, 204, 255));
		slider.addChangeListener(new ChangeListener() {
			public void stateChanged(ChangeEvent arg0) {
				String sliderNumber = sliderFormat.format((slider.getValue()) * .01);
				if (sliderNumber.length() == 3) {
					sliderNumber += "0";
				} else if (sliderNumber.equals("0")) {
					sliderNumber = "0.00";
				} else if (sliderNumber.equals("1")) {
					sliderNumber = "1.00";
				}
				lable.setText(lableName.substring(0, lableName.indexOf(":") + 2) + sliderNumber);
			}
		});// End of ChangeListener
		sliderBox.add(lable);
		sliderBox.add(slider);
		box.add(sliderBox);
		box.add(Box.createHorizontalGlue());
		return slider;
	}// End of createJSliders

	private JButton makeButtons(String iconFile, String lableName, int scaleWidth, int scaleHeight, boolean createGlue,
			JComponent group, boolean isIcon, ActionListener listener) {
		JButton button = new JButton();
		if (isIcon) {
			ImageIcon icon = new ImageIcon("C:/Users/Kyle Peters/Documents/picturesForPaintAppProject/" + iconFile);
			Image image = icon.getImage();
			image = image.getScaledInstance(scaleWidth, scaleHeight, Image.SCALE_SMOOTH);
			icon = new ImageIcon(image);
			button.setIcon(icon);
		} else {
			button.setText(lableName);
		}
		button.addActionListener(listener);
		button.setBackground(Color.WHITE);
		button.setCursor(new Cursor(Cursor.HAND_CURSOR));

		if (createGlue) {
			Box box = Box.createVerticalBox();
			JLabel label = new JLabel(lableName);
			label.setHorizontalAlignment(JLabel.CENTER);
			label.setFont(font);
			box.add(label);
			box.add(button);
			group.add(box);
			group.add(Box.createHorizontalGlue());
		} else {
			group.add(button);
		}
		return button;

	}// End of makeButtons

	private JTextField createTextField(int x, JComponent component) {
		JTextField field = new JTextField(x);
		component.add(field);
		return field;
	}// End of createTextField

	private JMenuItem createItems(String text, JMenu editMenu2, String iconName, boolean makeTool) {
		JMenuItem item = new JMenuItem(text);

		ImageIcon icon = new ImageIcon("C:/Users/Kyle Peters/Documents/picturesForPaintAppProject/" + iconName);
		Image image = icon.getImage();
		image = image.getScaledInstance(20, 20, Image.SCALE_SMOOTH);
		icon = new ImageIcon(image);

		item.setIcon(icon);
		item.setFont(new Font("Serif", Font.PLAIN, 15));
		MouseListener defaultMenuListen = item.getMouseListeners()[0];
		MouseListener listen = new MouseListener() {
			public void mouseClicked(MouseEvent e) {
				defaultMenuListen.mouseClicked(e);
			}
			public void mouseEntered(MouseEvent e) {
			}
			public void mouseExited(MouseEvent e) {
				defaultMenuListen.mouseExited(e);
			}
			public void mousePressed(MouseEvent e) {
				defaultMenuListen.mousePressed(e);
			}
			public void mouseReleased(MouseEvent e) {
				defaultMenuListen.mouseReleased(e);
			}
		};

		item.removeMouseListener(defaultMenuListen);
		item.addMouseListener(listen);
		item.addActionListener(menuListener);
		item.addMouseListener(mouseListener);
		editMenu2.add(item);
		if (makeTool) {
			makeTools(iconName, text.substring(0,text.indexOf("  ")));
		}
		return item;
	}// End of createItems

	private void createEditMenu() {
		editMenu = new JMenu("Edit");
		editMenu.setMnemonic(KeyEvent.VK_E);
		editMenu.addMenuListener(new MenuListener() {
			public void menuCanceled(MenuEvent e) {}
			public void menuDeselected(MenuEvent e) {
				addToToolBar.setBackground(editMenu.getBackground());
				deleteFromEdit.setBackground(editMenu.getBackground());
				gradient.setBackground(editMenu.getBackground());
				changeTransparency.setBackground(editMenu.getBackground());
				selectAll.setBackground(editMenu.getBackground());
				changeColor.setBackground(editMenu.getBackground());
				cut.setBackground(editMenu.getBackground());
				copy.setBackground(editMenu.getBackground());
				paste.setBackground(editMenu.getBackground());
				redo.setBackground(editMenu.getBackground());
				undo.setBackground(editMenu.getBackground());
			}
			public void menuSelected(MenuEvent e) {}
		});// End of PopupMenuListener
		selectAll = createItems("selectAll  ", editMenu, " ", false);
		selectAll.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_A,ActionEvent.CTRL_MASK));
		gradient = createItems("Gradient  ", editMenu, "gradient.png", true);
		gradient.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_G,ActionEvent.CTRL_MASK));
		changeTransparency = createItems("Transparency  ", editMenu, "transparency.jpg",true);
		changeTransparency.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_T,ActionEvent.CTRL_MASK));
		deleteFromEdit = createItems("Delete  ", editMenu, "delete.png", true);
		deleteFromEdit.setAccelerator(KeyStroke.getKeyStroke("DELETE"));
		changeColor=createItems("Change Colors             ", editMenu, "colorChange.png", true);
		addToToolBar = createItems("ToolBar Addition  ", editMenu, "add.png", true);
		copy=createItems("Copy  ",editMenu,"copy.png",true);
		copy.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_C,ActionEvent.CTRL_MASK));
		setMapping(copy);
		paste=createItems("Paste  ",editMenu,"paste.png",true);
		paste.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_V,ActionEvent.CTRL_MASK));
		setMapping(paste);
		cut=createItems("Cut  ",editMenu,"cut.png",true);
		cut.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_X,ActionEvent.CTRL_MASK));
		setMapping(cut);
		redo=createItems("Redo  ",editMenu,"redo.png",true);
		redo.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_Y,ActionEvent.CTRL_MASK));
		undo=createItems("Undo  ",editMenu,"undo.png",true);
		undo.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_Z,ActionEvent.CTRL_MASK));
	}// End of createEditMenu
	private void setMapping(JMenuItem menuItem){
	    ActionMap map = menuItem.getActionMap();
        map.put(TransferHandler.getCutAction().getValue(Action.NAME),
                TransferHandler.getCutAction());
        map.put(TransferHandler.getCopyAction().getValue(Action.NAME),
                TransferHandler.getCopyAction());
        map.put(TransferHandler.getPasteAction().getValue(Action.NAME),
                TransferHandler.getPasteAction());
	}
	private void createFileMenu() {
		fileMenu = new JMenu("File");
		fileMenu.setMnemonic(KeyEvent.VK_F);
		fileMenu.addMenuListener(new MenuListener() {
			public void menuCanceled(MenuEvent e) {}
			public void menuDeselected(MenuEvent e) {
				addNewCanvas.setBackground(editMenu.getBackground());
				save.setBackground(editMenu.getBackground());
				saveAs.setBackground(editMenu.getBackground());
				openFile.setBackground(editMenu.getBackground());
				saveAll.setBackground(editMenu.getBackground());
			}
			public void menuSelected(MenuEvent e) {	}
		});// End of PopupMenuListener
		addNewCanvas = createItems("New  ", fileMenu, "", false);
		addNewCanvas.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_N,ActionEvent.ALT_MASK+ActionEvent.SHIFT_MASK));
		openFile=createItems("Open File             ", fileMenu, "folderIcon.png", true);
		openFile.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_F,ActionEvent.ALT_MASK+ActionEvent.SHIFT_MASK));
		save = createItems("Save  ", fileMenu, "saveIcon.png", true);
		save.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_S,ActionEvent.CTRL_MASK));
		saveAs = createItems("Save As  ", fileMenu, "saveAsIcon.png", true);
		saveAs.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_S,ActionEvent.CTRL_MASK+ActionEvent.SHIFT_MASK));
		saveAll=createItems("Save All  ",fileMenu,"saveAllIcon.png",true);
		saveAll.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_A,ActionEvent.CTRL_MASK+ActionEvent.SHIFT_MASK));
	}// End of createFileMenu

	private void makeTools(String iconFile, String lableName) {
		JButton button = new JButton();
		ImageIcon icon = new ImageIcon("C:/Users/Kyle Peters/Documents/picturesForPaintAppProject/" + iconFile);
		Image image = icon.getImage();
		image = image.getScaledInstance(20, 20, Image.SCALE_SMOOTH);
		icon = new ImageIcon(image);
		button.setIcon(icon);
		button.addActionListener(menuListener);
		buttonMap.put(lableName, button);
		JCheckBox checkBox=null;
		try {
			resultSet.next();
			checkBox = new JCheckBox(lableName,resultSet.getBoolean(3));
		} catch (SQLException e) {
			e.printStackTrace();
		}
		checkBoxList.add(checkBox);

	}//End of makeTools
	
	private void modifyEditMenu() {
		if ((paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getSelectedList().isEmpty()&&paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getImageSelectedList().isEmpty())||popupVisible) {
			gradient.setForeground(Color.GRAY);
			changeTransparency.setForeground(Color.GRAY);
			deleteFromEdit.setForeground(Color.GRAY);
			changeColor.setForeground(Color.GRAY);
			cut.setForeground(Color.GRAY);
			copy.setForeground(Color.GRAY);
		}else if(paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getSelectedList().isEmpty()){
			gradient.setForeground(Color.GRAY);
			changeColor.setForeground(Color.GRAY);
			changeTransparency.setForeground(Color.BLACK);
			deleteFromEdit.setForeground(Color.BLACK);
			cut.setForeground(Color.BLACK);
			copy.setForeground(Color.BLACK);
		}
		else {
			changeColor.setForeground(Color.BLACK);
			gradient.setForeground(Color.BLACK);
			changeTransparency.setForeground(Color.BLACK);
			deleteFromEdit.setForeground(Color.BLACK);
			cut.setForeground(Color.BLACK);
			copy.setForeground(Color.BLACK);
		}
		if(paintAppList.get(multipleCanvas.getSelectedIndex()).getMementoIndex()>=1){
			undo.setForeground(Color.BLACK);
		}else{
			undo.setForeground(Color.GRAY);
		}
		if(paintAppList.get(multipleCanvas.getSelectedIndex()).getMementoNumber()-1>=paintAppList.get(multipleCanvas.getSelectedIndex()).getMementoIndex()){
			redo.setForeground(Color.BLACK);
		}else{
			redo.setForeground(Color.GRAY);
		}
		try {
			Clipboard clipBoard =Toolkit.getDefaultToolkit().getSystemClipboard();
			Transferable transfer=clipBoard.getContents(paintAppList.get(multipleCanvas.getSelectedIndex()));
			BufferedImage image=(BufferedImage) transfer.getTransferData(DataFlavor.imageFlavor);
			paste.setForeground(Color.BLACK);
		} catch (UnsupportedFlavorException | IOException e) {
			paste.setForeground(Color.GRAY);
		}catch(Exception e){
			paste.setForeground(Color.GRAY);	
		}
		
	}// End of showEditMenu
	private void modifyFileMenu() {
		if (paintAppList.get(multipleCanvas.getSelectedIndex()).allReadySaved) {
			save.setForeground(Color.GRAY);
		} else {
			save.setForeground(Color.BLACK);
		}
		for(PaintApp instance:paintAppList){
			if(!instance.allReadySaved){
				saveAll.setForeground(Color.BLACK);
				break;
			}
			saveAll.setForeground(Color.GRAY);
		}
	}// End of showFileMenu
	private void setBooleans(boolean line, boolean ellipse, boolean rectangle, boolean brush, boolean eraser,
			boolean select, boolean move) {
		lineSelected = line;
		rectangleSelected = rectangle;
		ellipseSelected = ellipse;
		brushSelected = brush;
		eraserOn = eraser;
		if (!eraserOn) {
			imagePanel.setCursor(new Cursor(Cursor.DEFAULT_CURSOR));
		}
		selected = select;
		moveSelected = move;
	}// End of setBooleans
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	private void deleteSelected() {	
		for (paintCompoenent s : paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getSelectedList()) {
			paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getShapeList().remove(s);
		}
		for(paintCompoenent i:paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getImageSelectedList()){
			paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getImageList().remove(i);
		}
		paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().repaint();
		paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getSelectedList().removeAll(paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getSelectedList());
		paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getImageSelectedList().removeAll(paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getImageSelectedList());
		paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().setEraserOrder();
	
		paintAppList.get(multipleCanvas.getSelectedIndex()).allReadySaved=false;
	}// End of deleteSelected
	private void createAddToolbarFrame() {
		ToolBarAdd = new JFrame();
		ToolBarAdd.setTitle("Add to ToolBar");
		ToolBarAdd.setSize(300, 350);
		ToolBarAdd.setDefaultCloseOperation(JFrame.DISPOSE_ON_CLOSE);
		ToolBarAdd.setAlwaysOnTop(true);
		ToolBarAdd.setResizable(false);
		ToolBarAdd.setVisible(true);
		ToolBarAdd.setLayout(new GridBagLayout());
		constraints = new GridBagConstraints();
		createToolbarLayout(ToolBarAdd, constraints);

	}// End of createAddToolbarFrame

	private void createToolbarLayout(JFrame frame, GridBagConstraints constraints) {
		JLabel northLabel = new JLabel("Add Tools");
		northLabel.setFont(font);
		northLabel.setOpaque(true);
		northLabel.setForeground(new Color(51, 155, 255));
		northLabel.setBackground(new Color(153, 204, 255));
		JPanel northPanel = new JPanel();
		northPanel.setOpaque(true);
		northPanel.setBackground(new Color(153, 204, 255));
		northPanel.add(northLabel);

		Box box = Box.createVerticalBox();
		for (JCheckBox checkBox : checkBoxList) {	
				box.add(checkBox);
		}
		apply = new JButton("Apply");
		apply.addActionListener(menuListener);
		box.add(apply);
		JScrollPane pane = new JScrollPane(box, JScrollPane.VERTICAL_SCROLLBAR_AS_NEEDED,
				JScrollPane.HORIZONTAL_SCROLLBAR_NEVER);

		addWithGridBag(1, 1, 1, 1, 0, 0, GridBagConstraints.BOTH, GridBagConstraints.CENTER, constraints, frame,
				northPanel);

		addWithGridBag(1, 2, 1, 1, 100, 100, GridBagConstraints.BOTH, GridBagConstraints.NORTHEAST, constraints,
				frame, pane);
	}// End of createToolbarLayout
	private void createGradient() {
		try {
			ArrayList<paintCompoenent> list=new ArrayList<paintCompoenent>();
			list.addAll( paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getShapeList());
			list.addAll(paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getImageList());
			paintAppList.get(multipleCanvas.getSelectedIndex()).allReadySaved=false;
			for (paintCompoenent s : paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getSelectedList()) {
				if (!isHorizontal) {
					paint = new GradientPaint((int) s.getShape().getBounds2D().getX(), 0, firstColor.getBackground(),
							(int) ((Integer.parseInt(xPos.getText())) + s.getShape().getBounds2D().getX()), 0,
							secondColor.getBackground(), gradientInMiddle.isSelected());
					shapeClass shapeC=new shapeClass(s.getShape(),s.getTransparency(),s.getFillColor(),s.getStrokeColor(),s.getThickness(),paint);
					paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().setModifierOrder(s,shapeC,true,list);
				} else {
					GradientPaint paint = new GradientPaint(0, (int) s.getShape().getBounds2D().getY(),
							firstColor.getBackground(), 0,
							(float) ((Integer.parseInt(xPos.getText())) + s.getShape().getBounds().getY()),
							secondColor.getBackground(), gradientInMiddle.isSelected());
					shapeClass shapeC=new shapeClass(s.getShape(),s.getTransparency(),s.getFillColor(),s.getStrokeColor(),s.getThickness(),paint);
					paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().setModifierOrder(s,shapeC,true,list);
				}			
			}
			paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().setModifierOrder(null,null,false,list);
			colorFrame.dispose();
		} catch (NullPointerException | NumberFormatException ex) {
			JOptionPane.showMessageDialog(null, "you didn't enter an integer", "Error", JOptionPane.ERROR_MESSAGE);
		}
	}// End of createGradient

	private void createGradientPopup() {
		createColorPanel();
		colorFrame = new JFrame();
		colorFrame.addMouseListener(mouseListener);
		colorFrame.setSize(550, 500);
		colorFrame.setDefaultCloseOperation(JFrame.DISPOSE_ON_CLOSE);
		colorFrame.setAlwaysOnTop(true);
		colorFrame.setResizable(false);
		colorFrame.setVisible(true);
		colorFrame.setLayout(new GridBagLayout());
		constraints = new GridBagConstraints();
		createGradientLayout();
	}// End of createGradientPopup

	private void createGradientLayout() {
		addWithGridBag(1, 1, 2, 1, 50, 50, GridBagConstraints.BOTH, GridBagConstraints.NORTH, constraints,
				colorFrame, colorPanel);
		Box redBox = Box.createHorizontalBox();
		redBox.add(redL);
		red = createTextField(10, redBox);
		addWithGridBag(1, 2, 1, 1, 50, 50, GridBagConstraints.HORIZONTAL, GridBagConstraints.CENTER, constraints,
				colorFrame, redBox);
		Box greenBox = Box.createHorizontalBox();
		greenBox.add(greenL);
		green = createTextField(10, greenBox);
		addWithGridBag(1, 3, 1, 1, 50, 50, GridBagConstraints.HORIZONTAL, GridBagConstraints.CENTER, constraints,
				colorFrame, greenBox);
		Box blueBox = Box.createHorizontalBox();
		blueBox.add(blueL);
		blue = createTextField(10, blueBox);
		addWithGridBag(1, 4, 1, 1, 50, 50, GridBagConstraints.HORIZONTAL, GridBagConstraints.CENTER, constraints,
				colorFrame, blueBox);
		selectedColorHolder = new JPanel();
		Dimension d = new Dimension(100, 100);
		selectedColorHolder.setPreferredSize(d);
		selectedColorHolder.setMinimumSize(d);
		selectedColorHolder.setMaximumSize(d);
		selectedColorHolder.setBackground(Color.GRAY);
		addWithGridBag(2, 2, 1, 3, 50, 50, GridBagConstraints.NONE, GridBagConstraints.WEST, constraints,
				colorFrame, selectedColorHolder);
		Box vertOrHori = Box.createHorizontalBox();
		addHorizontal = makeButtons("", "Add Horizontal Gradients", 10, 10, false, vertOrHori, false, menuListener);
		addVertical = makeButtons("", "Add Vertical Gradients", 10, 10, false, vertOrHori, false, menuListener);
		addWithGridBag(1, 5, 2, 1, 50, 50, GridBagConstraints.NONE, GridBagConstraints.CENTER, constraints,
				colorFrame, vertOrHori);
		holdSpace = Box.createHorizontalBox();
		addWithGridBag(1, 6, 1, 1, 50, 50, GridBagConstraints.HORIZONTAL, GridBagConstraints.CENTER, constraints,
				colorFrame, holdSpace);
	}// End of createGradientLayout

	private Box addGradientCriteria(String s) {
		Dimension d = new Dimension(30, 26);
		Box box = Box.createHorizontalBox();
		setFirst = makeButtons("", "Set First Color", 10, 10, false, box, false, menuListener);
		firstColor = new JPanel();
		firstColor.setBackground(Color.GRAY);
		firstColor.setPreferredSize(d);
		firstColor.setMinimumSize(d);
		firstColor.setMaximumSize(d);
		box.add(firstColor);
		xPos = new JTextField(s);
		xPos.addFocusListener(new FocusListener() {
			public void focusGained(FocusEvent e) {
				if (xPos.getText().equals(s)) {
					xPos.setText("");
				}
			}// end of FocusGained

			public void focusLost(FocusEvent e) {
				if (xPos.getText().equals("")) {
					xPos.setText(s);
				}
			}// end of focusLost
		});
		box.add(xPos);
		setSecond = makeButtons("", "Set Second Color", 10, 10, false, box, false, menuListener);
		secondColor = new JPanel();
		secondColor.setBackground(Color.GRAY);
		secondColor.setPreferredSize(d);
		secondColor.setMinimumSize(d);
		secondColor.setMaximumSize(d);
		box.add(secondColor);
		gradientInMiddle = new JCheckBox("Middle Gradient");
		box.add(gradientInMiddle);
		createGradient = makeButtons("", "Create", 10, 10, false, box, false, menuListener);
		return box;
	}// End of addHorizontalGradient

	private void changeTransparency() {
		try {
			Float newTrans = Float
					.parseFloat(JOptionPane.showInputDialog(null, "enter your new transparency between 0 and 1"));
			if (newTrans <= 1 && newTrans >= 0) {
				ArrayList<paintCompoenent> list=new ArrayList<paintCompoenent>();
				list.addAll(paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getShapeList());
				list.addAll(paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getImageList());
				for (paintCompoenent s : paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getSelectedList()) {
					shapeClass shapeC=new shapeClass(s.getShape(),newTrans,s.getFillColor(),s.getStrokeColor(),s.getThickness(),s.getPaint());
					paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().setModifierOrder(s,shapeC,true,list);
				}
				paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().setModifierOrder(null,null,false,list);
				for(paintCompoenent i:paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getImageSelectedList()){
					i.setTransparency(newTrans);
				}
				paintAppList.get(multipleCanvas.getSelectedIndex()).allReadySaved=false;
			} else {
				JOptionPane.showMessageDialog(null, "you didn't enter an integer between 0 and 1", "Error",
						JOptionPane.ERROR_MESSAGE);
			}
		} catch (NumberFormatException exception) {
			JOptionPane.showMessageDialog(null, "you didn't enter an integer", "Error", JOptionPane.ERROR_MESSAGE);
		} catch (NullPointerException exception) {

		}
	}// End of changeTransparency

	

	private void saveProjectAs() {
		String home=System.getProperty("user.home");
		JLabel label=(JLabel) deletePanels.get(multipleCanvas.getSelectedIndex()).getComponent(0);
		File file1=new File(home+"\\Documents\\"+label.getText());
		
		JFileChooser chooser=new JFileChooser();
		chooser.setSelectedFile(file1);
		
		int result=chooser.showSaveDialog(paintApp);
		chooser.setFileSelectionMode(JFileChooser.DIRECTORIES_ONLY);
		Dimension dimesnsion=Toolkit.getDefaultToolkit().getScreenSize();
				int x=(int) (dimesnsion.getWidth()/2-chooser.getWidth()/2);
				int y=(int) (dimesnsion.getHeight()/2-chooser.getHeight()/2);
		chooser.setLocation(x,y);
		File file = chooser.getSelectedFile();
		File projectToSave=null;
		int overide=0;
		if(result==JFileChooser.APPROVE_OPTION){
			try {
				projectToSave=new File(file.getPath()+".png");
				if(projectToSave.exists()){
					overide=JOptionPane.showConfirmDialog(chooser, "The file all ready exists, do you wish To overide it");
				}
				if(overide==0){
					projectToSave.delete();
				}else if(overide==1||overide==2){
					saveProjectAs();
				}
				projectToSave.createNewFile();
				BufferedImage bufferedImage=createBufferedImage();
				ImageIO.write(bufferedImage, "png", projectToSave);
			} catch (IOException|NullPointerException e) {
				JOptionPane.showMessageDialog(null, "Save didn't work","Error",JOptionPane.ERROR_MESSAGE);
				return;
			}
			if(!filenameList.isEmpty()){
				filenameList.get(multipleCanvas.getSelectedIndex()).delete();
				filenameList.remove(multipleCanvas.getSelectedIndex());
				filenameList.add(multipleCanvas.getSelectedIndex(),projectToSave);
			}
		}
	}//End of saveProjectAs
	
	private BufferedImage createBufferedImage(){
		BufferedImage bufferedImage=new BufferedImage(imagePanel.getWidth(),imagePanel.getHeight(), BufferedImage.TYPE_INT_RGB);
		Graphics2D graphics=bufferedImage.createGraphics();
		graphics.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
		graphics.setColor(Color.WHITE);
		graphics.fill(new Rectangle2D.Double(0,0,imagePanel.getWidth(),imagePanel.getHeight()));
		GradientPaint paint=null;
		for (paintCompoenent s: paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getShapeList()) {
			graphics.setStroke(new BasicStroke(s.getThickness()));
			graphics.setPaint(s.getStrokeColor());
			graphics.setComposite(AlphaComposite.getInstance(AlphaComposite.SRC_OVER, s.getTransparency()));
			graphics.draw(s.getShape());
			try{
				paint = s.getPaint();
			}catch(NullPointerException e){
				
			}
			if (paint != null) {
				graphics.setPaint(paint);
			} else {
				graphics.setPaint(s.getFillColor());
			}
			graphics.fill(s.getShape());
		}
		for(paintCompoenent i:paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getImageList()){
			graphics.setComposite(AlphaComposite.getInstance(AlphaComposite.SRC_OVER, i.getTransparency()));
			graphics.drawImage(i.getImage(), i.getX(), i.getY(), i.getWidth(),i.getHeight(), paintAppList.get(multipleCanvas.getSelectedIndex()));
		}
		graphics.dispose();
		return bufferedImage;

	}//End of createBufferedImage
	private void openFile(){
		paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().setCursor(new Cursor(Cursor.WAIT_CURSOR));
		JFileChooser fileChooser=new JFileChooser();
		FileFilter imageFilter = new FileNameExtensionFilter("Image files", ImageIO.getReaderFileSuffixes());
		fileChooser.setFileFilter(imageFilter);
		fileChooser.showOpenDialog(paintApp);
		File fileToOpen=fileChooser.getSelectedFile();
			BufferedImage image=null;
			try {
				image = ImageIO.read(fileToOpen);
			} catch (IOException|IllegalArgumentException e) {
			return;
			}
			imageClass imageC=new imageClass(image,1,0,0,image.getWidth(paintApp),image.getHeight(paintApp));
			paintAppList.get(multipleCanvas.getSelectedIndex()).allReadySaved=false;
			paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getImageList().add(imageC);
			paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().setCursor(new Cursor(Cursor.DEFAULT_CURSOR));
			paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().setImageOrder(imageC);
			paintAppList.get(multipleCanvas.getSelectedIndex()).allReadySaved=false;
			
	}//End of openFile
	private void apply(){
		for (JCheckBox checkBox : checkBoxList) {
			if (checkBox.isSelected()) {
				toolboxList.add(buttonMap.get(checkBox.getText()));
				try {
	resultSet.absolute(checkBoxList.indexOf(checkBox)+1);	resultSet.updateBoolean(3,true);   resultSet.updateRow();
				} catch (SQLException e1){}
				}
			else if (!checkBox.isSelected() && toolboxList.contains(buttonMap.get(checkBox.getText()))) {
				toolboxList.remove(buttonMap.get(checkBox.getText()));
				try {
	resultSet.absolute(checkBoxList.indexOf(checkBox)+1);  resultSet.updateBoolean(3,false);  resultSet.updateRow();
				} catch (SQLException e1){}
			}}
		createToolbar(false);
		reCreateLayout();
		ToolBarAdd.dispose();
	}//End of Apply
	private void save(){
	if(!paintAppList.get(multipleCanvas.getSelectedIndex()).allReadySaved){
		try {
			BufferedImage bufferedImage=createBufferedImage();
			ImageIO.write(bufferedImage, "png", filenameList.get(multipleCanvas.getSelectedIndex()));
		} catch (IOException e) {
			JOptionPane.showMessageDialog(null, "save didn't work");
			return;
		}
		paintAppList.get(multipleCanvas.getSelectedIndex()).allReadySaved=true;
	}
	}//End of Save
	private void saveAll() {
		for(PaintApp instance:paintAppList){
			if(!instance.allReadySaved){
				try {
					BufferedImage bufferedImage=createBufferedImage();
					ImageIO.write(bufferedImage, "png", filenameList.get(paintAppList.indexOf(instance)));
				} catch (IOException e) {
					JOptionPane.showMessageDialog(null, "save didn't work");
					return;
				}
				instance.allReadySaved=true;
			}		
		}
	}//End of saveAll;
	private void changeColor(){
		if(!paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getSelectedList().isEmpty()){
		createColorPanel();
		colorFrame = new JFrame();
		colorFrame.addMouseListener(mouseListener);
		colorFrame.setSize(550, 500);
		colorFrame.setDefaultCloseOperation(JFrame.DISPOSE_ON_CLOSE);
		colorFrame.setAlwaysOnTop(true);
		colorFrame.setResizable(false);
		colorFrame.setVisible(true);
		colorFrame.setLayout(new GridBagLayout());
		constraints = new GridBagConstraints();
		JButton fill=new JButton("Apply To Fill");
		fill.addActionListener(new ActionListener(){
			public void actionPerformed(ActionEvent e) {
				ArrayList<paintCompoenent> list=new ArrayList<paintCompoenent>();
				list.addAll(paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getShapeList());
				list.addAll(paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getImageList());
				for(paintCompoenent s:paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getSelectedList()){
					shapeClass shapeC=new shapeClass(s.getShape(),s.getTransparency(),selectedColorHolder.getBackground(),s.getStrokeColor(),s.getThickness(),s.getPaint());
					paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().setModifierOrder(s,shapeC,true,list);
				}
				paintAppList.get(multipleCanvas.getSelectedIndex()).allReadySaved=false;
				colorFrame.dispose();
				paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().setModifierOrder(null,null,false,list);
					repaint();
			}});//End of actionListener
		JButton stroke=new JButton("Apply To Stroke");
		stroke.addActionListener(new ActionListener(){
			public void actionPerformed(ActionEvent e) {
				ArrayList<paintCompoenent> list=new ArrayList<paintCompoenent>();
				list.addAll(paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getShapeList());
				list.addAll(paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getImageList());
				for(paintCompoenent s:paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getSelectedList()){
					shapeClass shapeC=new shapeClass(s.getShape(),s.getTransparency(),s.getFillColor(),selectedColorHolder.getBackground(),s.getThickness(),s.getPaint());
					paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().setModifierOrder(s,shapeC,true,list);
				}
				paintAppList.get(multipleCanvas.getSelectedIndex()).allReadySaved=false;
				paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().setModifierOrder(null,null,false,list);
				repaint();	
				colorFrame.dispose();
			}});//End of actionListener
		createColorChangeLayout();
		addWithGridBag(1, 5, 1, 1, 50, 50, GridBagConstraints.NONE, GridBagConstraints.EAST, constraints,colorFrame, fill);
		addWithGridBag(2, 5, 1, 1, 50, 50, GridBagConstraints.NONE, GridBagConstraints.WEST, constraints,colorFrame, stroke);
	}}//End of changeColors
	private void createColorChangeLayout() {
		addWithGridBag(1, 1, 2, 1, 50, 50, GridBagConstraints.BOTH, GridBagConstraints.NORTH, constraints,colorFrame, colorPanel);
		Box redBox = Box.createHorizontalBox();
		redBox.add(redL);
		red = createTextField(10, redBox);
		addWithGridBag(1, 2, 1, 1, 50, 50, GridBagConstraints.HORIZONTAL, GridBagConstraints.CENTER, constraints,
				colorFrame, redBox);
		Box greenBox = Box.createHorizontalBox();
		greenBox.add(greenL);
		green = createTextField(10, greenBox);
		addWithGridBag(1, 3, 1, 1, 50, 50, GridBagConstraints.HORIZONTAL, GridBagConstraints.CENTER, constraints,
				colorFrame, greenBox);
		Box blueBox = Box.createHorizontalBox();
		blueBox.add(blueL);
		blue = createTextField(10, blueBox);
		addWithGridBag(1, 4, 1, 1, 50, 50, GridBagConstraints.HORIZONTAL, GridBagConstraints.CENTER, constraints,
				colorFrame, blueBox);
		selectedColorHolder = new JPanel();
		Dimension d = new Dimension(100, 100);
		selectedColorHolder.setPreferredSize(d);
		selectedColorHolder.setMinimumSize(d);
		selectedColorHolder.setMaximumSize(d);
		selectedColorHolder.setBackground(Color.GRAY);
		addWithGridBag(2, 2, 1, 3, 50, 50, GridBagConstraints.NONE, GridBagConstraints.WEST, constraints,
				colorFrame, selectedColorHolder);
	}//End of createColorChangeLayout
	private void pasteToCanvas() {
		Clipboard clipBoard =Toolkit.getDefaultToolkit().getSystemClipboard();
		Transferable transfer=clipBoard.getContents(paintAppList.get(multipleCanvas.getSelectedIndex()));
		if(transfer.isDataFlavorSupported(DataFlavor.imageFlavor)&&transfer!=null){
			try {
				BufferedImage image=(BufferedImage) transfer.getTransferData((DataFlavor.imageFlavor));
				BufferedImage image2=new BufferedImage(image.getWidth(),image.getHeight(),BufferedImage.TYPE_INT_RGB);
				Graphics2D graphics=image2.createGraphics();
				graphics.drawImage(image, image.getMinX(), image.getMinY(),paintAppList.get(multipleCanvas.getSelectedIndex()));
				Point p=paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getPastePosition();
				imageClass imageC=null;
				if(p!=null){
					imageC=new imageClass(image2,1F,(int)p.getX(),(int)p.getY(),image2.getWidth(),image2.getHeight());
				}else{
					imageC=new imageClass(image2,1F,0,0,image2.getWidth(),image2.getHeight());
				}
				paintAppList.get(multipleCanvas.getSelectedIndex()).allReadySaved=false;
				paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getImageList().add(imageC);
				paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().setImageOrder(imageC);
			}catch (UnsupportedFlavorException | IOException e) {
				JOptionPane.showMessageDialog(null, "Paste Not Supported","Error",JOptionPane.ERROR_MESSAGE);
			}
		}else{
			
		}	
		graphics.dispose();
	}// End of pasteToCanvas
	private void copy(){
		try{
		Clipboard clipBoard =Toolkit.getDefaultToolkit().getSystemClipboard();
		ImageTransferable transferable = new ImageTransferable(getCopiedImage());
		clipBoard.setContents(transferable,null);
		}catch(NullPointerException e){
			
		}
	}//End of copy
	private void cut(){
		try{
			Clipboard clipBoard =Toolkit.getDefaultToolkit().getSystemClipboard();
			ImageTransferable transferable = new ImageTransferable(getCopiedImage());
			clipBoard.setContents(transferable,null);
			}catch(NullPointerException e){
				
			}
		deleteSelected();
	}//End of cut
	private BufferedImage getCopiedImage(){
		if(!paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getSelectedList().isEmpty()||!paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getImageSelectedList().isEmpty()){
		int minX=10000,minY=1000,maxX=-1,maxY=-1;
		for(paintCompoenent s:paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getSelectedList()){
			if(s.getShape().getBounds2D().getMinX()<minX)minX=(int) s.getShape().getBounds2D().getMinX();
			if(s.getShape().getBounds2D().getMinY()<minY)minY=(int) s.getShape().getBounds2D().getMinY();
			if(s.getShape().getBounds2D().getMaxX()>maxX)maxX=(int) s.getShape().getBounds2D().getMaxX();
			if(s.getShape().getBounds2D().getMaxY()>maxY)maxY=(int) s.getShape().getBounds2D().getMaxY();
		}
		for(paintCompoenent i:paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getImageSelectedList()){
			if(i.getX()<minX)minX=i.getX();
			if(i.getY()<minY)minY=i.getY();
			if(i.getWidth()>maxX)maxX=i.getWidth();
			if(i.getHeight()>maxY)maxY=i.getHeight();
		}
		if(maxX!=-1){
			BufferedImage bufferedImage=new BufferedImage(maxX-minX,maxY-minY,BufferedImage.TYPE_INT_RGB);
			Graphics2D graphics=bufferedImage.createGraphics();
			graphics.setPaint(Color.WHITE);
			graphics.fillRect(0,0,maxX-minX,maxY-minY);
			for(paintCompoenent s:paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getSelectedList()){
				graphics.setPaint(s.getStrokeColor());
				graphics.setComposite(AlphaComposite.getInstance(AlphaComposite.SRC_OVER,s.getTransparency()));
				graphics.setStroke(new BasicStroke(s.getThickness()));
				if(s.getShape().getClass().toString().contains("Line2D")){
					Line2D line=(Line2D)s.getShape();
					Line2D line2=new Line2D.Double(line.getP1().getX()-minX,line.getP1().getY()-minY, line.getP2().getX()-minX, line.getP2().getY()-minY);
					graphics.draw(line2);
				}else if(s.getShape().getClass().toString().contains("Rectangle2D")){
					Rectangle2D rec=(Rectangle2D)s.getShape();
					//Rectangle2D rec2=new Rectangle2D.Double(rec.getMinX()-minX,rec.getMinY()-minY, rec.getMaxX()-minX, rec.getMaxY()-minY);
					Rectangle2D rec2=new Rectangle2D.Double(rec.getMinX()-minX,rec.getMinY()-minY, rec.getWidth(), rec.getHeight());
					graphics.draw(rec2);
					if(s.getPaint()!=null){graphics.setPaint(s.getPaint());}
					else{graphics.setPaint(s.getFillColor());}
					graphics.fill(rec2);
				}else if(s.getShape().getClass().toString().contains("Ellipse2D")){
					Ellipse2D ellipse=(Ellipse2D)s.getShape();
					Ellipse2D ellipse2=new Ellipse2D.Double(ellipse.getMinX()-minX,ellipse.getMinY()-minY, ellipse.getWidth(), ellipse.getHeight());	
					if(s.getPaint()!=null){graphics.setPaint(s.getPaint());}
					else{graphics.setPaint(s.getFillColor());}
					graphics.fill(ellipse2);
				}
			}
			for(paintCompoenent i:paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getImageSelectedList()){
				graphics.setComposite(AlphaComposite.getInstance(AlphaComposite.SRC_OVER,i.getTransparency()));
				graphics.drawImage(i.getImage(),i.getX()-minX,i.getY()-minY,i.getWidth()-minX,i.getHeight()-minY,paintAppList.get(multipleCanvas.getSelectedIndex()));
			}	
			graphics.dispose();
			return bufferedImage;
			}	
		}
		return null;
	}//End of getCopiedImage
	
	
	
	
	
	
	
	
	
	
	private class buttonListen implements ActionListener {
		public void actionPerformed(ActionEvent e) {
			if (e.getSource() == fColor) {
				fillColor = JColorChooser.showDialog(null, "Color Chooser", fillColor);
				if (fillColor == null) {
					fillColor = Color.BLACK;
				}
			} else if (e.getSource() == sColor) {
				strokeColor = JColorChooser.showDialog(null, "Color Chooser", strokeColor);
				if (strokeColor == null) {
					strokeColor = Color.BLACK;
				}
			} else if (e.getSource() == rectangle) {
				setBooleans(false, false, true, false, false, false, false);
			} else if (e.getSource() == line) {
				setBooleans(true, false, false, false, false, false, false);
			} else if (e.getSource() == brush) {
				setBooleans(false, false, false, true, false, false, false);
			} else if (e.getSource() == ellipse) {
				setBooleans(false, true, false, false, false, false, false);
			} else if (e.getSource() == select) {
				setBooleans(false, false, false, false, false, true, false);
			} else if (e.getSource() == eraser) {
				Toolkit toolkit = Toolkit.getDefaultToolkit();
				ImageIcon icon = new ImageIcon(
						"C:/Users/Kyle Peters/Documents/picturesForPaintAppProject/eraser.jpg");
				imagePanel.setCursor(toolkit.createCustomCursor(icon.getImage(), new Point(1, 1), "Custom Cursor"));
				setBooleans(false, false, false, false, true, false, false);
			} else if (e.getSource() == delete) {
				deleteSelected();
			} else if (e.getSource() == move) {
				setBooleans(false, false, false, false, false, selected, true);
			}
		}// End of actionPerformed
	}// End of buttonListen Class
	
	
	private class menuListen implements ActionListener {
		public void actionPerformed(ActionEvent e) {
			if (e.getSource() == deleteFromEdit || e.getSource() == buttonMap.get("Delete")) {
				if (!paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getSelectedList().isEmpty()||!paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getImageSelectedList().isEmpty()) {
					deleteSelected();
				}
			} else if (e.getSource() == gradient || e.getSource() == buttonMap.get("Gradient")) {popupVisible=true;
				if (!paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getSelectedList().isEmpty()) {
					createGradientPopup();
				}
			} else if (e.getSource() == changeTransparency || e.getSource() == buttonMap.get("Transparency")) {
				if (!paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getSelectedList().isEmpty()||!paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getImageSelectedList().isEmpty()) {
					changeTransparency();
				}
			} else if (e.getSource() == selectAll || e.getSource() == buttonMap.get("selectAll")) {
				selected = true;lineSelected = false;ellipseSelected = false;brushSelected = false;eraserOn = false;moveSelected = false;
				paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getSelectedList().removeAll(paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getSelectedList());
				paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().setSelectedList(paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getShapeList());
				paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getImageSelectedList().removeAll(paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getImageSelectedList());
				paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().setImageSelectedList(paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getImageList());
			} else if (e.getSource() == addHorizontal) {
				isHorizontal = true;
				colorFrame.remove(holdSpace);
				addWithGridBag(1, 6, 2, 1, 50, 50, GridBagConstraints.BOTH, GridBagConstraints.CENTER, constraints,colorFrame, holdSpace = addGradientCriteria("XPos"));
				colorFrame.revalidate();
			} else if (e.getSource() == addVertical) {
				isHorizontal = false;
				colorFrame.remove(holdSpace);
				addWithGridBag(1, 6, 2, 1, 50, 50, GridBagConstraints.BOTH, GridBagConstraints.CENTER, constraints,colorFrame, holdSpace = addGradientCriteria("YPos"));
				colorFrame.revalidate();
			}else if (e.getSource() == setFirst)firstColor.setBackground(selectedColorHolder.getBackground());
			else if (e.getSource() == setSecond)secondColor.setBackground(selectedColorHolder.getBackground());
			else if (e.getSource() == createGradient) createGradient();
			else if (e.getSource() == addToToolBar || e.getSource() == buttonMap.get("ToolBar Addition"))createAddToolbarFrame();
			else if (e.getSource() == apply)apply();
			else if(e.getSource()==addNewCanvas){
				String home=System.getProperty("user.home");
				String number=(filenameList.isEmpty())?"":Integer.toString(filenameList.size());
				File file=new File(home+"\\Downloads\\canvas"+number+".png");
				int number2;
				if(!number.equals("")){number2=Integer.parseInt(number);}
				else{number2=0;}
				while(file.exists()){
					number2++;
				file=new File(home+"\\Downloads\\canvas"+number2+".png");
				}
				try {
					file.createNewFile();
				} catch (IOException e1) {
					// TODO Auto-generated catch block
					e1.printStackTrace();
				}
				filenameList.add(file);
				paintAppList.add(new PaintApp(false,"canvas"+number2));
			}
			else if(e.getSource()==saveAs|| e.getSource() == buttonMap.get("Save As"))saveProjectAs();
			else if(e.getSource()==openFile||e.getSource()== buttonMap.get("Open File"))openFile();
			else if(e.getSource()==save|| e.getSource() == buttonMap.get("Save"))save();
			else if(e.getSource()==saveAll|| e.getSource() == buttonMap.get("Save All"))saveAll();
			else if(e.getSource()==changeColor|| e.getSource() == buttonMap.get("Change Colors"))changeColor();
			else if(e.getSource()==paste||e.getSource() == buttonMap.get("Paste"))pasteToCanvas();
			else if(e.getSource()==copy||e.getSource() == buttonMap.get("Copy"))copy();
			else if(e.getSource()==cut||e.getSource()==buttonMap.get("Cut"))cut();
			else if(e.getSource()==undo||e.getSource()==buttonMap.get("Undo"))undo();
			else if(e.getSource()==redo||e.getSource()==buttonMap.get("Redo"))redo();
			paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().repaint();
		}// End of ActionPerformed
		private void undo(){
			if(paintAppList.get(multipleCanvas.getSelectedIndex()).getMementoIndex()>=1){
				paintAppList.get(multipleCanvas.getSelectedIndex()).getUndoIndexes().add(careTaker.getMemento(paintAppList.get(multipleCanvas.getSelectedIndex()).getMementoIndex()));
				paintAppList.get(multipleCanvas.getSelectedIndex()).setMementoIndex(-1);
				list=originator.restoreFromMemento(careTaker.getMemento(paintAppList.get(multipleCanvas.getSelectedIndex()).getMementoIndex()));
				ArrayList<paintCompoenent> tempList=new ArrayList<paintCompoenent>();
				tempList.addAll(list);

				paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getShapeList().removeAll(paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getShapeList());
				paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getImageList().removeAll(paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getImageList());
				for(paintCompoenent p:tempList){
					if(p.getImage()==null){
						paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getShapeList().add(p);
					}else{
						paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getImageList().add(p);
					}
				}
				paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getSelectedList().removeAll(paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getSelectedList());
				paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getImageSelectedList().removeAll(paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getImageSelectedList());
			}
		}
		private void redo(){
		
			if(paintAppList.get(multipleCanvas.getSelectedIndex()).getMementoNumber()>paintAppList.get(multipleCanvas.getSelectedIndex()).getMementoIndex()&&paintAppList.get(multipleCanvas.getSelectedIndex()).getMementoNumber()<=careTaker.getMementoListSize()+1){	
				try{
				paintAppList.get(multipleCanvas.getSelectedIndex()).getUndoIndexes().remove(careTaker.getMemento(paintAppList.get(multipleCanvas.getSelectedIndex()).getMementoIndex()));
				paintAppList.get(multipleCanvas.getSelectedIndex()).setMementoIndex(1);
				paintAppList.get(multipleCanvas.getSelectedIndex()).getUndoIndexes().remove(careTaker.getMemento(paintAppList.get(multipleCanvas.getSelectedIndex()).getMementoIndex()));
				}catch(IndexOutOfBoundsException e){
					paintAppList.get(multipleCanvas.getSelectedIndex()).setMementoIndex(1);
				}
				
				list=(originator.restoreFromMemento(careTaker.getMemento(paintAppList.get(multipleCanvas.getSelectedIndex()).getMementoIndex())));
			
				paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getShapeList().removeAll(paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getShapeList());
				paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getImageList().removeAll(paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getImageList());
				for(paintCompoenent p:list){
					if(p.getImage()==null){
						paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getShapeList().add(p);
					}else{
						paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getImageList().add(p);
					}
				}
				paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getSelectedList().remove(paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getSelectedList());
				paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getImageSelectedList().removeAll(paintAppList.get(multipleCanvas.getSelectedIndex()).getImagePanel().getImageSelectedList());			}
		}
		
	}// End of menuListener









	private class mouseListen implements MouseListener {

		@Override
		public void mouseClicked(MouseEvent e) {
			 if(e.getSource()==multipleCanvas){
					int x=110*multipleCanvas.getSelectedIndex()-103;
			if(e.getX()>x&&e.getX()<x+120&&e.getY()>5&&e.getY()<17){
				multipleCanvas.remove(multipleCanvas.getSelectedIndex());
			}
				 return;
			 }
			
			for (JButton b : colorButtonHolder) {
				if (e.getSource() == b) {
					red.setText(Integer.toString(b.getBackground().getRed()));
					green.setText(Integer.toString(b.getBackground().getGreen()));
					blue.setText(Integer.toString(b.getBackground().getBlue()));
					selectedColorHolder.setBackground(b.getBackground());
				}
			}
		}// End of mouseClicked

		@Override
		public void mouseEntered(MouseEvent e) {
			setBackground(addToToolBar.getForeground(), e, addToToolBar, false);
			setBackground(changeColor.getForeground(), e, changeColor, false);
			setBackground(saveAll.getForeground(), e, saveAll, false);
			setBackground(deleteFromEdit.getForeground(), e, deleteFromEdit, false);
			setBackground(gradient.getForeground(), e, gradient, false);
			setBackground(changeTransparency.getForeground(), e, changeTransparency, false);
			setBackground(selectAll.getForeground(), e, selectAll, false);
			setBackground(paste.getForeground(), e, paste, false);
			setBackground(copy.getForeground(), e, copy, false);
			setBackground(cut.getForeground(), e, cut, false);
			setBackground(undo.getForeground(), e, undo, false);
			setBackground(redo.getForeground(), e, redo, false);
			setBackground(addNewCanvas.getForeground(), e, addNewCanvas, false);
			setBackground(save.getForeground(), e, save, false);
			setBackground(saveAs.getForeground(), e, saveAs, false);
			setBackground(openFile.getForeground(), e, openFile, false);
		}// End of mouseEntered

		private void setBackground(Color check, MouseEvent e, JMenuItem itemToCheck, boolean mouseExited) {
			if (e.getSource() == itemToCheck) {
				if (mouseExited) {
					itemToCheck.setBackground(editMenu.getBackground());
				} else {
					if (check == Color.GRAY) {
						itemToCheck.setBackground(new Color(192, 192, 192));
					} else {
						itemToCheck.setBackground(new Color(51, 152, 255));
					}
				}
			}
		}//end of setBackground

		@Override
		public void mouseExited(MouseEvent e) {
			setBackground(addToToolBar.getForeground(), e, addToToolBar, true);
			setBackground(changeColor.getForeground(), e, changeColor, true);
			setBackground(saveAll.getForeground(), e, saveAll, true);
			setBackground(deleteFromEdit.getForeground(), e, deleteFromEdit, true);
			setBackground(gradient.getForeground(), e, gradient, true);
			setBackground(changeTransparency.getForeground(), e, changeTransparency, true);
			setBackground(selectAll.getForeground(), e, selectAll, true);
			setBackground(paste.getForeground(), e, paste, true);
			setBackground(copy.getForeground(), e, copy, true);
			setBackground(cut.getForeground(), e, cut, true);
			setBackground(undo.getForeground(), e, undo, true);
			setBackground(redo.getForeground(), e, redo, true);
			setBackground(addNewCanvas.getForeground(), e, addNewCanvas, true);
			setBackground(save.getForeground(), e, save, true);
			setBackground(saveAs.getForeground(), e, saveAs, true);
			setBackground(openFile.getForeground(), e, openFile, true);
		}// End of mouseExited

		@Override
		public void mousePressed(MouseEvent e) {
			for (JButton b : colorButtonHolder) {
				if (e.getSource() == b) {
					red.setText(Integer.toString(b.getBackground().getRed()));
					green.setText(Integer.toString(b.getBackground().getGreen()));
					blue.setText(Integer.toString(b.getBackground().getBlue()));
					selectedColorHolder.setBackground(b.getBackground());
				}
			}
		}

		@Override
		public void mouseReleased(MouseEvent e) {

		}

	}// End of mouseListen

	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	private class image extends JComponent {
		private ArrayList<paintCompoenent> tempShapeList = new ArrayList<paintCompoenent>();
		private ArrayList<paintCompoenent> tempImageList = new ArrayList<paintCompoenent>();
		private frameClickListener mouseListener = new frameClickListener();
		private int maxX, maxY, minX, minY;
		private int startX, startY, endX, endY, moveStartX, moveStartY;
		private Color strokeBlue = new Color(51, 255, 255);
		private Color fillBlue = new Color(0, 128, 255);
		private boolean drawShadow = false, moveShapes = false, shapeHitEdge = false;
		private Rectangle2D moveButton, resizeButton;
		private Image moveIcon;
		private ArrayList<paintCompoenent> shapeList, selectedList;
		private ArrayList<paintCompoenent>imageSelectedList,imageList;
		private boolean topSelected,rightSelected,leftSelected,bottomSelected,resizeSelected;
		private Shape moveShape,resizeShape;
		private Image moveImage;
		private int forMoveImage;
		private Rectangle resizeImage;
		private Ellipse2D resizeLine1,resizeLine2;
		private Line2D resizeLine;
		private boolean lineP1Resize,lineP2Resize;
		private ArrayList <paintCompoenent> erasedList;
		private Point pastePosition;
		private ArrayList<shapeClass> brushLists;
		private Point2D point1,point2;
		public ArrayList<paintCompoenent> getImageList() {
			return imageList;
		}
		public void setImageSelectedList(ArrayList<paintCompoenent> imageSelectedList) {
			this.imageSelectedList.addAll(imageSelectedList);
			
		}
		public ArrayList<paintCompoenent> getImageSelectedList() {
			return imageSelectedList;
		}
	
		public Point getPastePosition() {
			return pastePosition;
		}
		private ArrayList<paintCompoenent> getSelectedList(){
			return selectedList;
		}
		private void setSelectedList(ArrayList<paintCompoenent>selectedList){
			this.selectedList.addAll(selectedList);
		}
		private ArrayList<paintCompoenent> getShapeList(){
			return shapeList;
		}

		public image() {
			erasedList=new ArrayList <paintCompoenent>();
			brushLists=new ArrayList<shapeClass>();
			selectedList = new ArrayList<paintCompoenent>();
			shapeList = new ArrayList<paintCompoenent>();
			imageList=new ArrayList<paintCompoenent>();
			imageSelectedList=new ArrayList<paintCompoenent>();
			this.addMouseListener(mouseListener);
			ArrayList<paintCompoenent> list=new ArrayList<paintCompoenent>();
			this.addMouseMotionListener(mouseListener);
			mementoIndex=careTaker.getMementoListSize();
			mementoNumber=careTaker.getMementoListSize();
			originator.set(list);
			careTaker.addMemento(originator.storeMemento());	
		}//End of constructor image

		public void paint(Graphics g) {
			graphics = (Graphics2D) g;

			graphics.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
			drawShapes(graphics);
			if(!imageList.isEmpty()){
				for(paintCompoenent i:imageList){
					graphics.setComposite(AlphaComposite.getInstance(AlphaComposite.SRC_OVER, i.getTransparency()));
					graphics.drawImage(i.getImage(),i.getX(),i.getY(),i.getWidth(),i.getHeight(), this);
				}
			}
			if (moveIcon != null) {
				graphics.setStroke(new BasicStroke(1));
				graphics.drawImage(moveIcon, (int) moveButton.getBounds2D().getX(),
						(int) moveButton.getBounds2D().getY(), this);
				if(resizeButton!=null){
					graphics.draw(resizeButton);
					graphics.setComposite(AlphaComposite.getInstance(AlphaComposite.SRC_OVER, .2F));
					graphics.setPaint(fillBlue);
					graphics.fill(resizeButton);
				}else if(resizeLine1!=null &&  resizeLine2!=null){
					graphics.setComposite(AlphaComposite.getInstance(AlphaComposite.SRC_OVER, .5F));
					graphics.setPaint(fillBlue);
					graphics.fill(resizeLine1);
					graphics.fill(resizeLine2);
				}
			}
			if (drawShadow) {
				drawShadow(graphics);
			}
			this.revalidate();
			graphics.dispose();
		}

		private void drawShapes(Graphics2D graphics) {
			try {

				GradientPaint paint=null;
				for (int x = 0; x < shapeList.size(); x++) {

					graphics.setStroke(new BasicStroke(shapeList.get(x).getThickness()));
					graphics.setPaint(shapeList.get(x).getStrokeColor());
					graphics.setComposite(AlphaComposite.getInstance(AlphaComposite.SRC_OVER, shapeList.get(x).getTransparency()));
					graphics.draw(shapeList.get(x).getShape());
					
					try{
					paint = (shapeList.get(x).getPaint());
					}catch(NullPointerException e){}
					if (paint != null) {
						graphics.setPaint(paint);
					} else {
						graphics.setPaint(shapeList.get(x).getFillColor());
					}
					graphics.fill(shapeList.get(x).getShape());
				}
				if (!selectedList.isEmpty() && selected) {
					graphics.setStroke(new BasicStroke(10));
					for (paintCompoenent s : selectedList) {
						graphics.setComposite(AlphaComposite.getInstance(AlphaComposite.SRC_OVER, 0.2F));
						graphics.setPaint(strokeBlue);
						graphics.draw(s.getShape());
						graphics.setPaint(fillBlue);
						graphics.fill(s.getShape());
					}
				}
				if(!imageSelectedList.isEmpty()&&selected){
					graphics.setStroke(new BasicStroke(10));
					for (paintCompoenent i:imageSelectedList){
						graphics.setComposite(AlphaComposite.getInstance(AlphaComposite.SRC_OVER, 0.2F));
						graphics.setPaint(strokeBlue);
						graphics.drawImage(i.getImage(),i.getX(),i.getY(),i.getWidth(),i.getHeight(), this);
						graphics.setPaint(fillBlue);
						graphics.fillRect(i.getX()-10,i.getY()-10,i.getWidth()+20,i.getHeight()+20);

					}
				}
			} catch (NullPointerException | IndexOutOfBoundsException e) {

			}
		}//End of drawShapes

		
		private void drawShadow(Graphics2D graphics) {
			graphics.setComposite(AlphaComposite.getInstance(AlphaComposite.SRC_OVER, 0.2F));
			graphics.setPaint(Color.GRAY);
			maxX = Math.max(startX, endX);
			maxY = Math.max(startY, endY);
			minX = Math.min(startX, endX);
			minY = Math.min(startY, endY);
			if (rectangleSelected) {
				graphics.fill(new Rectangle2D.Double(minX, minY, maxX - minX, maxY - minY));
			} else if (lineSelected) {
				graphics.setStroke(new BasicStroke(thickness.getValue()));
				if(endY<0){
				endX = (endX> paintApp.getWidth()) ? paintApp.getWidth() : endX;
				endY = (endY > paintApp.getHeight()) ? paintApp.getHeight() : endY;
				endX = ( endX < 0) ? 0 : endX;
				endY = ( endY < 0) ? 0 : endY;}
				graphics.draw(new Line2D.Double(startX, startY, endX, endY));
			} else if (ellipseSelected) {
				graphics.fill(new Ellipse2D.Double(minX, minY, maxX - minX, maxY - minY));
			} else if (selected) {
				graphics.setStroke(new BasicStroke(10));
				graphics.setComposite(AlphaComposite.getInstance(AlphaComposite.SRC_OVER, 0.4F));
				graphics.setPaint(strokeBlue);
				graphics.draw(new Rectangle2D.Double(minX, minY, maxX - minX, maxY - minY));
				graphics.setPaint(fillBlue);
				graphics.fill(new Rectangle2D.Double(minX, minY, maxX - minX, maxY - minY));
			}
			drawShadow = false;
		}

		private void setOrder(paintCompoenent s){
				ArrayList<paintCompoenent> list=new ArrayList<paintCompoenent>();
				list.addAll(imageList);
				list.addAll(shapeList);
				shapeClass shapeC=new shapeClass(s.getShape(),s.getTransparency(),s.getFillColor(),s.getStrokeColor(),s.getThickness(),s.getPaint());
					int index=list.indexOf(s);
				list.remove(index);
				list.add(index,shapeC);
				mementoIndex=careTaker.getMementoListSize();
				mementoNumber=careTaker.getMementoListSize();
				originator.set(list);
				careTaker.addMemento(originator.storeMemento());
				if(careTaker.getMementoListSize()>50){
					careTaker.removeIndex(0);
					mementoIndex--;
					mementoNumber--;
				}
			if(!paintAppList.get(multipleCanvas.getSelectedIndex()).getUndoIndexes().isEmpty()){
				
				for(Memento x:paintAppList.get(multipleCanvas.getSelectedIndex()).getUndoIndexes()){
					careTaker.removeIndex(x);
				}	
				mementoIndex=careTaker.getMementoListSize()-1;
				mementoNumber=careTaker.getMementoListSize()-1;
				paintAppList.get(multipleCanvas.getSelectedIndex()).getUndoIndexes().removeAll(paintAppList.get(multipleCanvas.getSelectedIndex()).getUndoIndexes());
				}
		}
		private void setImageOrder(paintCompoenent s){
					ArrayList<paintCompoenent> list=new ArrayList<paintCompoenent>();
					list.addAll(imageList);
					list.addAll(shapeList);
					imageClass shapeC=new imageClass(s.getImage(),s.getTransparency(),s.getX(),s.getY(),s.getWidth(),s.getHeight());
						int index=list.indexOf(s);
					list.remove(index);
					list.add(index,shapeC);
					mementoIndex=careTaker.getMementoListSize();
					mementoNumber=careTaker.getMementoListSize();
					originator.set(list);
					careTaker.addMemento(originator.storeMemento());
					if(careTaker.getMementoListSize()>50){
						careTaker.removeIndex(0);

						mementoIndex--;
						mementoNumber--;
					}
				if(!paintAppList.get(multipleCanvas.getSelectedIndex()).getUndoIndexes().isEmpty()){
					
					for(Memento x:paintAppList.get(multipleCanvas.getSelectedIndex()).getUndoIndexes()){
						careTaker.removeIndex(x);
					}	
					mementoIndex=careTaker.getMementoListSize()-1;
					mementoNumber=careTaker.getMementoListSize()-1;
					paintAppList.get(multipleCanvas.getSelectedIndex()).getUndoIndexes().removeAll(paintAppList.get(multipleCanvas.getSelectedIndex()).getUndoIndexes());
					}
		}
		private void setModifierOrder(paintCompoenent s,paintCompoenent s2,boolean setShape,ArrayList<paintCompoenent> list){
			if(setShape){
			int index=list.indexOf(s);
			list.remove(index);
			list.add(index,s2);
			int x=shapeList.indexOf(s);
			shapeList.remove(s);
			shapeList.add(x,s2);
			}else{
				mementoIndex=careTaker.getMementoListSize();
				mementoNumber=careTaker.getMementoListSize();
				originator.set(list);
				careTaker.addMemento(originator.storeMemento());
			if(careTaker.getMementoListSize()>50){
				careTaker.removeIndex(0);
				mementoIndex--;
				mementoNumber--;
			}
			}
if(!paintAppList.get(multipleCanvas.getSelectedIndex()).getUndoIndexes().isEmpty()){
				
				for(Memento x:paintAppList.get(multipleCanvas.getSelectedIndex()).getUndoIndexes()){
					careTaker.removeIndex(x);
				}	
				mementoIndex=careTaker.getMementoListSize();
				mementoNumber=careTaker.getMementoListSize();
				paintAppList.get(multipleCanvas.getSelectedIndex()).getUndoIndexes().removeAll(paintAppList.get(multipleCanvas.getSelectedIndex()).getUndoIndexes());
			}
		}
		private ArrayList<paintCompoenent> setBrushOrder(paintCompoenent s){
			ArrayList<paintCompoenent> list=new ArrayList<paintCompoenent>();
			list.addAll(shapeList);
			list.addAll(imageList);
			shapeClass shapeC=new shapeClass(s.getShape(),s.getTransparency(),s.getFillColor(),s.getStrokeColor(),s.getThickness(),s.getPaint());
				int index=list.indexOf(s);
			list.remove(index);
			list.add(index,shapeC);
		if(!paintAppList.get(multipleCanvas.getSelectedIndex()).getUndoIndexes().isEmpty()){
			
			for(Memento x:paintAppList.get(multipleCanvas.getSelectedIndex()).getUndoIndexes()){
				careTaker.removeIndex(x);
			}	
			mementoIndex=careTaker.getMementoListSize();
			mementoNumber=careTaker.getMementoListSize();
			paintAppList.get(multipleCanvas.getSelectedIndex()).getUndoIndexes().removeAll(paintAppList.get(multipleCanvas.getSelectedIndex()).getUndoIndexes());
		}
		return list;
		}
		private void setEraserOrder(){
			ArrayList<paintCompoenent> list=new ArrayList<paintCompoenent>();
			list.addAll(shapeList);
			list.addAll(imageList);
			mementoIndex=careTaker.getMementoListSize();
			mementoNumber=careTaker.getMementoListSize();
			originator.set(list);
			careTaker.addMemento(originator.storeMemento());
			if(careTaker.getMementoListSize()>50){
				careTaker.removeIndex(0);

				mementoIndex--;
				mementoNumber--;
			}
			if(!paintAppList.get(multipleCanvas.getSelectedIndex()).getUndoIndexes().isEmpty()){
				
				for(Memento x:paintAppList.get(multipleCanvas.getSelectedIndex()).getUndoIndexes()){
					careTaker.removeIndex(x);
				}	
				mementoIndex=careTaker.getMementoListSize();
				mementoNumber=careTaker.getMementoListSize();
				paintAppList.get(multipleCanvas.getSelectedIndex()).getUndoIndexes().removeAll(paintAppList.get(multipleCanvas.getSelectedIndex()).getUndoIndexes());
			}
		}
		
		
		
		
		
		
		
		
		
		
		
		
		
		
		
		
		
		
		private class frameClickListener implements MouseListener, MouseMotionListener {
			public void mouseDragged(MouseEvent e) {
				paintAppList.get(multipleCanvas.getSelectedIndex()).allReadySaved = false;
				if (brushSelected) {
					shapeClass shapeC=new shapeClass(new Ellipse2D.Double(e.getX()-thickness.getValue()/2, e.getY()-thickness.getValue()/2, thickness.getValue(), thickness.getValue()),(float) (transparent.getValue() * .01),fillColor,strokeColor,1,null);
					shapeList.add(shapeC);
					brushLists.add(shapeC);
				} else if (eraserOn) {
					tempShapeList.addAll(shapeList);
					for (paintCompoenent s : tempShapeList) {
						if (s.getShape().intersects(e.getX() - 10, e.getY() - 10, 50, 50)) {
							shapeList.remove(s);
							erasedList.add(s);
						}
					}
					tempShapeList.removeAll(tempShapeList);
					tempImageList.addAll(imageList);
					for (paintCompoenent i:tempImageList) {
						if (new Rectangle(i.getX(),i.getY(),i.getWidth(),i.getHeight()).intersects(e.getX() - 10, e.getY() - 10, 50, 50)) {
							imageList.remove(i);
							erasedList.add(i);
						}
					}
					tempImageList.removeAll(tempImageList);
				}else if(resizeSelected){
					if(topSelected){
						if(resizeImage==null)moveTop(e,resizeShape,false);
						else moveTop(e,resizeImage,true);
					}else if(bottomSelected){
						if(resizeImage==null)moveBottom(e,resizeShape,false);
						else moveBottom(e,resizeImage,true);
					}
					if(rightSelected){
						if(resizeImage==null)moveRight(e,resizeShape,false);
						else moveRight(e,resizeImage,true);
					}else if(leftSelected){
						if(resizeImage==null)moveLeft(e,resizeShape,false);
						else moveLeft(e,resizeImage,true);
					}
				}else if(lineP1Resize){
					int x=e.getX();
					int y=e.getY();
					x=(x<0)?0:x;  x=(x>paintAppList.get(multipleCanvas.getSelectedIndex()).getWidth())?paintAppList.get(multipleCanvas.getSelectedIndex()).getWidth():x;
					y=(y<0)?0:y; y=(y>paintAppList.get(multipleCanvas.getSelectedIndex()).getHeight())?paintAppList.get(multipleCanvas.getSelectedIndex()).getHeight():y;
					resizeLine.setLine(x, y, point2.getX(),  point2.getY());
				}else if(lineP2Resize){
					int x=e.getX();
					int y=e.getY();
					x=(x<0)?0:x;  x=(x>paintAppList.get(multipleCanvas.getSelectedIndex()).getWidth())?paintAppList.get(multipleCanvas.getSelectedIndex()).getWidth():x;
					y=(y<0)?0:y; y=(y>paintAppList.get(multipleCanvas.getSelectedIndex()).getHeight())?paintAppList.get(multipleCanvas.getSelectedIndex()).getHeight():y;
					resizeLine.setLine(point1.getX(), point1.getY(),x, y);
				}
				else if (moveSelected) {
					moveShape(e);
				} else {

					endX = e.getX();
					endY = e.getY();
					drawShadow = true;
				}
				repaint();
			}// End of mouseDragged
			public void mouseMoved(MouseEvent e) {
				if(resizeButton!=null){
					int secondX=(int) (resizeButton.getBounds2D().getX()+resizeButton.getBounds2D().getWidth());
					int secondY=(int) (resizeButton.getBounds2D().getY()+resizeButton.getBounds2D().getHeight());
					if((new Rectangle((int)resizeButton.getBounds2D().getX()-5,(int)resizeButton.getBounds2D().getY()-5,10,10)).intersects(e.getX(),e.getY(),1,1)){
						setSelected(false,true,true,false);imagePanel.setCursor(new Cursor(Cursor.NW_RESIZE_CURSOR));
					}else if((new Rectangle(secondX-5,(int)resizeButton.getBounds2D().getY()-5,10,10)).intersects(e.getX(),e.getY(),1,1)){
						setSelected(true,false,true,false);imagePanel.setCursor(new Cursor(Cursor.NE_RESIZE_CURSOR));
					}else if((new Rectangle((int)resizeButton.getBounds2D().getX()-5,secondY-5,10,10)).intersects(e.getX(),e.getY(),1,1)){
						setSelected(false,true,false,true); imagePanel.setCursor(new Cursor(Cursor.SW_RESIZE_CURSOR));
					}else if((new Rectangle(secondX-5,secondY-5,10,10)).intersects(e.getX(),e.getY(),1,1)){
						setSelected(true,false,false,true); imagePanel.setCursor(new Cursor(Cursor.SE_RESIZE_CURSOR));
					}else if((new Rectangle((int)resizeButton.getBounds2D().getX()-5,(int)resizeButton.getBounds2D().getY(),10,(int)resizeButton.getBounds2D().getHeight()).intersects(e.getX(),e.getY(),1,1))){
						setSelected(false,true,false,false); imagePanel.setCursor(new Cursor(Cursor.E_RESIZE_CURSOR));
					}else if((new Rectangle(secondX-5,(int)resizeButton.getBounds2D().getY()-5,10,secondY).intersects(e.getX(),e.getY(),1,1))){
						setSelected(true,false,false,false); imagePanel.setCursor(new Cursor(Cursor.W_RESIZE_CURSOR));
					}else if((new Rectangle((int)resizeButton.getBounds2D().getX()-5,(int)resizeButton.getBounds2D().getY()-5,(int)resizeButton.getBounds2D().getWidth(),10).intersects(e.getX(),e.getY(),1,1))){
						setSelected(false,false,true,false); imagePanel.setCursor(new Cursor(Cursor.N_RESIZE_CURSOR));
					}else if((new Rectangle((int)resizeButton.getBounds2D().getX()-5,secondY,(int)resizeButton.getBounds2D().getWidth(),10).intersects(e.getX(),e.getY(),1,1))){
						setSelected(false,false,false,true); imagePanel.setCursor(new Cursor(Cursor.S_RESIZE_CURSOR));
					}else{
						setSelected(false,false,false,false);
						imagePanel.setCursor(new Cursor(Cursor.DEFAULT_CURSOR));
					
					}
				}
			}//end of mouseMoved


			public void mouseClicked(MouseEvent e) {
				pastePosition=new Point(e.getX(),e.getY());
				if (!moveSelected) {
					selectedList.removeAll(selectedList);
					imageSelectedList.removeAll(imageSelectedList);
				}
				paintAppList.get(multipleCanvas.getSelectedIndex()).allReadySaved = false;
				if (brushSelected) {
					shapeClass shapeC=new shapeClass(new Ellipse2D.Double(e.getX()-thickness.getValue()/2, e.getY()-thickness.getValue()/2, thickness.getValue(), thickness.getValue()),(float) (transparent.getValue() * .01),fillColor,strokeColor,1,null);
					shapeList.add(shapeC);
					shapeC.setFillColor(fillColor);
					shapeC.setStrokeColor(strokeColor);
					shapeC.setTransparency(((float) (transparent.getValue() * .01)));
				} else if (eraserOn) {
					tempShapeList.addAll(shapeList);
					for (paintCompoenent s : tempShapeList) {
						if (s.getShape().intersects(e.getX() - 10, e.getY() - 10, 50, 50)) {
							shapeList.remove(s);
							repaint();
						}
					}
					tempShapeList.removeAll(tempShapeList);
				}else if (moveSelected) {
					if (!selectedList.isEmpty()) {
						moveSelectedList(e);
					} else {
						moveOneShape(e);
					}
				}else if (selected) {
					for (paintCompoenent s : shapeList) {
						if (s.getShape().intersects(e.getX(), e.getY(), 1, 1) && !selectedList.contains(s)) {
							selectedList.add(s);
						}
					}
					for(paintCompoenent i:imageList){
						if(new Rectangle(i.getX(),i.getY(),i.getWidth(),i.getHeight()).intersects(e.getX(), e.getY(), 1, 1)&& !imageSelectedList.contains(i)){
							imageSelectedList.add(i);
						}
					}
				}
				repaint();
			}// End of MouseClicked

			public void mouseEntered(MouseEvent e) {

			}
			public void mouseExited(MouseEvent e) {
			}
			public void mousePressed(MouseEvent e) {	
				pastePosition=new Point(e.getX(),e.getY());
				if (moveSelected) {
					if (moveIcon != null && moveButton.intersects(e.getX(), e.getY(), 10, 10)) {
						if (!selectedList.isEmpty()) {
							if (moveButton.intersects(e.getX(), e.getY(), 10, 10)) {
								moveShapes = true;
								moveStartX = e.getX();
								moveStartY = e.getY();
								}else{
								moveShapes = false;
								moveIcon = null;
								moveButton = null;
								selectedList.removeAll(selectedList);
								imageSelectedList.removeAll(imageSelectedList);
								}
						} else {
							if (moveButton.intersects(e.getX(), e.getY(), 10, 10)) {
							moveShapes = true;
							moveStartX = e.getX();
							moveStartY = e.getY();
							} else {
							moveShapes = false;
							}
						}
					} 
					else {
						moveIcon=null;
					}
				} else if (!moveSelected) {
					moveIcon = null;
					selectedList.removeAll(selectedList);
					imageSelectedList.removeAll(imageSelectedList);
				}
				if(topSelected||rightSelected||bottomSelected||leftSelected){
					resizeSelected=true;
					selectedList.removeAll(selectedList);
					moveStartX = e.getX();
					moveStartY = e.getY();
				}
				startX = e.getX();
				startY = e.getY();
				try{
					 if(resizeLine1.intersects(e.getX(),e.getY(),1,1)){
						lineP1Resize=true;
					}else if(resizeLine2.intersects(e.getX(),e.getY(),1,1)){
						lineP2Resize=true;
					}
				}catch(NullPointerException exception){}
			}// End of mousePressed
			@Override
			public void mouseReleased(MouseEvent e) {
				int distanceX = 0, distanceY = 0;
				resizeSelected=false; resizeLine1=null;resizeLine2=null;resizeLine=null;
				resizeImage=null; resizeShape=null; lineP1Resize=false; lineP2Resize=false;
				shapeClass shapeC=null;
				if (!lineSelected) {
					minX = Math.min(startX, e.getX());
					minX = (minX < 0) ? 0 : minX;
					minY = Math.min(startY, e.getY());
					minY = (minY < 0) ? 0 : minY;

					maxX = Math.max(startX, e.getX());
					distanceX = (minX + (maxX - minX) > paintApp.getWidth()) ? paintApp.getWidth() - minX : maxX - minX;
					maxY = Math.max(startY, e.getY());
					distanceY = (minY + (maxY - minY) > imagePanel.getHeight()) ? imagePanel.getHeight() - minY
							: maxY - minY;
					
				}
				if (rectangleSelected) {
					shapeC=new shapeClass(new Rectangle2D.Double(minX, minY, distanceX, distanceY),(float) (transparent.getValue() * .01),fillColor,strokeColor,1,null);
					shapeList.add(shapeC);
					setOrder(shapeC);						
				} else if (lineSelected) {
					distanceX = (e.getX() > paintApp.getWidth()) ? paintApp.getWidth() : e.getX();
					distanceY = (e.getY() > paintApp.getHeight()) ? paintApp.getHeight() : e.getY();
					distanceX = (distanceX == e.getX() && e.getX() < 0) ? 0 : e.getX();
					distanceY = (distanceY == e.getY() && e.getY() < 0) ? 0 : e.getY();
					shapeC=new shapeClass(new Line2D.Double(startX, startY, distanceX, distanceY),(float) (transparent.getValue() * .01),fillColor,strokeColor,thickness.getValue(),null);
					shapeList.add(shapeC);
					setOrder(shapeC);
				} else if (ellipseSelected) {
					shapeC=new shapeClass(new Ellipse2D.Double(minX, minY, distanceX, distanceY),(float) (transparent.getValue() * .01),fillColor,strokeColor,1,null);
					shapeList.add(shapeC);
					setOrder(shapeC);
				}else if (selected) {
					for (paintCompoenent s : shapeList) {
						if (s.getShape().intersects(minX, minY, distanceX, distanceY) && !selectedList.contains(s)) {
							selectedList.add(s);
						}
					}
					for(paintCompoenent i:imageList){
						if(new Rectangle(i.getX(),i.getY(),i.getWidth(),i.getHeight()).intersects(minX, minY, distanceX, distanceY)&& !imageSelectedList.contains(i)){
							imageSelectedList.add(i);
						}
					}			
				}else if(eraserOn){
					setEraserOrder();
				}else if(brushSelected){
					ArrayList<paintCompoenent> shapeList=new ArrayList<paintCompoenent>();
					for(paintCompoenent shape:brushLists){
						shapeList=setBrushOrder(shape);
					}
					mementoIndex=careTaker.getMementoListSize();
					mementoNumber=careTaker.getMementoListSize();
					originator.set(shapeList);
					careTaker.addMemento(originator.storeMemento());
					brushLists.removeAll(brushLists);
					if(careTaker.getMementoListSize()>50){
						careTaker.removeIndex(0);
						mementoIndex--;
						mementoNumber--;
					}
				}else{
				return;
				}
				if(!selected&&!moveSelected&&!brushSelected&&!eraserOn){
					shapeC.setFillColor(fillColor);
					shapeC.setStrokeColor(strokeColor);
					shapeC.setTransparency((float) (transparent.getValue() * .01));
				}
				drawShadow = false;
				repaint();
			}

		}

		
		
		
		
		
		
		
		
		
		
		
		
		private void moveBottom(MouseEvent e,Shape shape,boolean isImage) {
			
			int x=(int) shape.getBounds2D().getX();
			int y=(int) shape.getBounds2D().getY();
			int width=(int) shape.getBounds2D().getWidth();
			int newHeight=e.getY()-y; newHeight=(newHeight+y>imagePanel.getHeight())?imagePanel.getHeight()-y:newHeight;	
			//Shape s=new Rectangle2D.Double(x,y,width,newHeight);
			if(isImage){
				imageList.get(forMoveImage).setX(x);
				imageList.get(forMoveImage).setY(y);
				imageList.get(forMoveImage).setWidth(width);
				imageList.get(forMoveImage).setHeight(newHeight);
			}else{
				if(shape.getClass().toString().contains("Rectangle2D")){
					Rectangle2D rec=(Rectangle2D)shape;
					rec.setRect(x,y,width,newHeight);
					resizeShape=rec;
				}else if(shape.getClass().toString().contains("Ellipse2D")){
					Ellipse2D rec=(Ellipse2D)shape;
					rec.setFrame(x, y, width, newHeight);
					resizeShape=rec;
				}
		}			
		}//End of moveBottom
		private void moveTop(MouseEvent e,Shape shape,boolean isImage){
			int x=(int) shape.getBounds2D().getX();
			int y=(int) shape.getBounds2D().getY();
			int width=(int) shape.getBounds2D().getWidth();
			int height=(int) shape.getBounds2D().getHeight();
			int distanceY=e.getY()-moveStartY;	
			int yPos=moveStartY+distanceY; yPos=(yPos<0)?0:yPos;
			if(isImage){
				imageList.get(forMoveImage).setX(x);
				imageList.get(forMoveImage).setY(yPos);
				imageList.get(forMoveImage).setWidth(width);
				imageList.get(forMoveImage).setHeight(height+(y-yPos));
			}else{
				if(shape.getClass().toString().contains("Rectangle2D")){
					Rectangle2D rec=(Rectangle2D)shape;
					rec.setRect(x,yPos,width,height+(y-yPos));
					resizeShape=rec;
				}else if(shape.getClass().toString().contains("Ellipse2D")){
					Ellipse2D rec=(Ellipse2D)shape;
					rec.setFrame(x,yPos,width,height+(y-yPos));
					resizeShape=rec;
				}
		}
		}//End of moveTop
		private void moveLeft(MouseEvent e,Shape shape,boolean isImage){
			int x=(int) shape.getBounds2D().getX();
			int y=(int) shape.getBounds2D().getY();
			int width=(int) shape.getBounds2D().getWidth();
			int height=(int) shape.getBounds2D().getHeight();
			int distanceX=e.getX()-moveStartX;	
			int xPos=moveStartX+distanceX; xPos=(xPos<0)?0:xPos;
			if(isImage){
				imageList.get(forMoveImage).setX(xPos);
				imageList.get(forMoveImage).setY(y);
				imageList.get(forMoveImage).setWidth(width+(x-xPos));
				imageList.get(forMoveImage).setHeight(height);
			}else{
				if(shape.getClass().toString().contains("Rectangle2D")){
					Rectangle2D rec=(Rectangle2D)shape;
					rec.setRect(xPos,y,width+(x-xPos),height);
					resizeShape=rec;
				}else if(shape.getClass().toString().contains("Ellipse2D")){
					Ellipse2D rec=(Ellipse2D)shape;
					rec.setFrame(xPos,y,width+(x-xPos),height);
					resizeShape=rec;
				}
		}
		}//End of moveLeft;
		private void moveRight(MouseEvent e,Shape shape,boolean isImage){
			int x=(int) shape.getBounds2D().getX();
			int y=(int) shape.getBounds2D().getY();
			int height=(int) shape.getBounds2D().getHeight();
			int newWidth=e.getX()-x; newWidth=(newWidth+x>imagePanel.getWidth())?imagePanel.getWidth()-x:newWidth;
			if(isImage){
				imageList.get(forMoveImage).setX(x);
				imageList.get(forMoveImage).setY(y);
				imageList.get(forMoveImage).setWidth(newWidth);
				imageList.get(forMoveImage).setHeight(height);
			}else{
				if(shape.getClass().toString().contains("Rectangle2D")){
					Rectangle2D rec=(Rectangle2D)shape;
					rec.setRect(x,y,newWidth,height);
					resizeShape=rec;
				}else if(shape.getClass().toString().contains("Ellipse2D")){
					Ellipse2D rec=(Ellipse2D)shape;
					rec.setFrame(x,y,newWidth,height);
					resizeShape=rec;
				}
			}
		}
		private void setSelected(boolean right,boolean left,boolean top,boolean bottom){
			topSelected=top;
			rightSelected=right;
			bottomSelected=bottom;
			leftSelected=left;
		}//End of setSelected
		
		
		private void moveShape(MouseEvent e) {
			if (moveIcon != null) {
				int newX = e.getX();
				int newY = e.getY();
				newX = (newX < 0) ? 0 : newX;
				newY = (newY < 0) ? 0 : newY;
				int distanceX = e.getX() - moveStartX;
				int distanceY = e.getY() - moveStartY;
				moveStartX += distanceX;
				moveStartY += distanceY;
				if (!selectedList.isEmpty()) {
					tempShapeList.removeAll(tempShapeList);
					tempShapeList.addAll(selectedList);
					if (!checkShapes(distanceX, distanceY, tempShapeList)) {
						for (paintCompoenent s : tempShapeList) {
							moveShapes(s.getShape(), distanceX, distanceY, true);
						}
					}
					if(moveShape==null){
						shapeHitEdge = false;
						tempShapeList.removeAll(tempShapeList);
						resizeButton=null;
					}else{
						moveShapes(moveShape, distanceX, distanceY, false);
					
						if(moveShape.getClass().toString().contains("Line2D")){
						resizeLine1=null;
						resizeLine2=null;
						}
						shapeHitEdge = false;
						tempShapeList.removeAll(tempShapeList);
						resizeButton=null;
					}
				} else if (moveShape!=null) {
					moveShapes(moveShape, distanceX, distanceY, true);
					moveShapes(moveShape, distanceX, distanceY, false);
					resizeButton=null;
					if(moveShape.getClass().toString().contains("Line2D")){
						resizeLine1=null;
						resizeLine2=null;
					}
				}
				
				
				if(!imageSelectedList.isEmpty()){
					tempImageList.removeAll(tempImageList);
					tempImageList.addAll(imageSelectedList);
					if (!checkImage(distanceX, distanceY, tempImageList)) {
						for (paintCompoenent i : tempImageList) {
							moveImages(i, distanceX, distanceY, true);
						}
					}
					if(forMoveImage!=10000){
					moveImages(imageList.get(forMoveImage), distanceX, distanceY, false);
					resizeButton=null;
					shapeHitEdge = false;
					tempImageList.removeAll(tempImageList);
					forMoveImage=10000;
					}else{
						resizeButton=null;
						shapeHitEdge = false;
						tempImageList.removeAll(tempImageList);
					}
				}else if (moveImage != null) {
					moveImages(imageList.get(forMoveImage), distanceX, distanceY, true);
					moveImages(imageList.get(forMoveImage), distanceX, distanceY, false);
					resizeButton=null;
				}
			}
		}// End of moveShape
		private boolean checkImage(int distanceX, int distanceY, ArrayList<paintCompoenent> temp) {
			boolean check = false;
			for (int index=0;index<temp.size();index++) {
				int topX = (int) temp.get(index).getX();
				int topY = (int) temp.get(index).getY();
				int width = (int) temp.get(index).getWidth();
				int height = (int) temp.get(index).getHeight();
				int x = (topX + distanceX < 0) ? 0 : topX + distanceX;
				int y = (topY + distanceY < 0) ? 0 : topY + distanceY;
				x = (x + width > paintApp.getWidth()) ? paintApp.getWidth() - width : x;
				y = (y + height > imagePanel.getHeight()) ? imagePanel.getHeight() - height : y;
				if (x == 0 || y == 0 || x == paintApp.getWidth() - width || y == imagePanel.getHeight() - height) {
					shapeHitEdge = true;
					break;
				}
			}
			return check;
		}// End of checkImage
	

		private boolean checkShapes(int distanceX, int distanceY, ArrayList<paintCompoenent> temp) {
			boolean check = false;
			for (paintCompoenent s : temp) {
				int topX = (int) s.getShape().getBounds2D().getX();
				int topY = (int) s.getShape().getBounds2D().getY();
				int width = (int) s.getShape().getBounds2D().getWidth();
				int height = (int) s.getShape().getBounds2D().getHeight();
				int x = (topX + distanceX < 0) ? 0 : topX + distanceX;
				int y = (topY + distanceY < 0) ? 0 : topY + distanceY;
				x = (x + width > paintApp.getWidth()) ? paintApp.getWidth() - width : x;
				y = (y + height > imagePanel.getHeight()) ? imagePanel.getHeight() - height : y;
				if (x == 0 || y == 0 || x == paintApp.getWidth() - width || y == imagePanel.getHeight() - height) {
					shapeHitEdge = true;
					break;
				}
			}
			return check;
		}// End of checkShapes
		
		
		
		private void moveImages(paintCompoenent i, int distanceX, int distanceY, boolean image) {
			if (moveShapes && !shapeHitEdge) {
				int topX = (int) i.getX();
				int topY = (int) i.getY();
				int width = (int) i.getWidth();
				int height = (int) i.getHeight();
				int x = (topX + distanceX < 0) ? 0 : topX + distanceX;
				int y = (topY + distanceY < 0) ? 0 : topY + distanceY;
				x = (x + width > paintApp.getWidth()) ? paintApp.getWidth() - width : x;
				y = (y + height > imagePanel.getHeight()) ? imagePanel.getHeight() - height : y;

				if (image) {
						//Rectangle rec=new Rectangle(i.getX(),i.getY(),i.getWidth(),i.getHeight());
					i.setX(x);
					i.setY(y);
					i.setWidth(width);
					i.setHeight(height);
				} else {
					moveButton.setRect((int) ((i.getX()) + (i.getWidth() / 2 - 20)),
							(int) (i.getY() - 40), 40, 40);
					
				}
			
			}
		}// End of makeShapes
		
		
		
		
		private void moveShapes(Shape s, int distanceX, int distanceY, boolean shape) {
			if (moveShapes && !shapeHitEdge) {
				int topX = (int) s.getBounds2D().getX();
				int topY = (int) s.getBounds2D().getY();
				int width = (int) s.getBounds2D().getWidth();
				int height = (int) s.getBounds2D().getHeight();
				int x = (topX + distanceX < 0) ? 0 : topX + distanceX;
				int y = (topY + distanceY < 0) ? 0 : topY + distanceY;
				x = (x + width > paintApp.getWidth()) ? paintApp.getWidth() - width : x;
				y = (y + height > imagePanel.getHeight()) ? imagePanel.getHeight() - height : y;
				if (shape) {
					if(s.getClass().toString().contains("Line2D")){
						Line2D line=(Line2D)s;
						line.setLine(line.getX1()+distanceX, line.getY1()+distanceY, line.getX2()+distanceX, line.getY2()+distanceY);
					}else if(s.getClass().toString().contains("Rectangle2D")){
						Rectangle2D rec=(Rectangle2D)s;
					rec.setRect(x, y, width, height);
					}else if(s.getClass().toString().contains("Ellipse2D")){
						Ellipse2D ellipse=(Ellipse2D)s;
					ellipse.setFrame(x, y, width, height);
					}
				} else {
					moveShape=s;
						moveButton.setRect((int) ((s.getBounds2D().getX()) + (s.getBounds2D().getWidth() / 2 - 20)),(int) (s.getBounds2D().getY() - 40), 40, 40);		
				}
			}
		}// End of makeShapes
		private void moveSelectedList(MouseEvent e){
			boolean intersects = false;
			for (paintCompoenent s : selectedList) {
				if (s.getShape().intersects(e.getX(), e.getY(), 10, 10)) {
					intersects = true;
					moveShape=s.getShape();
					createShapeOutline(s);
				}
			}
			for(paintCompoenent i:imageSelectedList){
				if (new Rectangle(i.getX(),i.getY(),i.getWidth(),i.getHeight()).intersects(e.getX(), e.getY(), 10, 10)) {
					intersects = true;
					moveImage=i.getImage();
					forMoveImage=imageList.indexOf(i);

					resizeImage=new Rectangle(i.getX(),i.getY(),i.getWidth(),i.getHeight());
					createImageOutline(new Rectangle(i.getX(),i.getY(),i.getWidth(),i.getHeight()));
				}		
			}
			if(!intersects){
			moveShapes = false;
			moveIcon = null;
			moveButton = null;
			selectedList.removeAll(selectedList);
			imageSelectedList.removeAll(imageSelectedList);
			}
	
		}//End of moveSelectedList
		private void moveOneShape(MouseEvent e){
			try {
				for (int x = shapeList.size() - 1; x >= 0; x--) {
					if (shapeList.get(x).getShape().intersects(e.getX(), e.getY(), 10, 10)) {
						moveShape = shapeList.get(x).getShape();
						createShapeOutline(shapeList.get(x));
						break;
					}
				}
				for(paintCompoenent i:imageList){
					if (new Rectangle(i.getX(),i.getY(),i.getWidth(),i.getHeight()).intersects(e.getX(), e.getY(), 10, 10)) {
						moveImage=i.getImage();
						forMoveImage=imageList.indexOf(i);
						resizeImage=new Rectangle(i.getX(),i.getY(),i.getWidth(),i.getHeight());
						createImageOutline(new Rectangle(i.getX(),i.getY(),i.getWidth(),i.getHeight()));
						break;
					}		
				}
			} catch (NullPointerException ex) {
				moveIcon = null;
				moveButton = null;
			}
		}//End of moveOneShape
		
		
		
		private void createImageOutline(Rectangle bounds) {
			ImageIcon icon = new ImageIcon("C:/Users/Kyle Peters/Documents/picturesForPaintAppProject/moveCursor.png");
			moveIcon = icon.getImage();
			moveIcon = moveIcon.getScaledInstance(40, 40, Image.SCALE_SMOOTH);
			int y=(int) (bounds.getY() - 40); y=(y<0)?0:y;
				moveButton = new Rectangle2D.Double(
						(int) ((bounds.getX()) + (bounds.getWidth() / 2 - 20)),y, 40, 40);
			
			int topX=(int) bounds.getX()-10; 
			int topY=(int) bounds.getY()-10;
			int width=(int) bounds.getWidth()+20;
			int height=(int) bounds.getHeight()+20;
			resizeButton = new Rectangle2D.Double(topX,topY,width,height);	
		}//End of createImageOutline
		
		private void createShapeOutline(paintCompoenent s) {
			ImageIcon icon = new ImageIcon("C:/Users/Kyle Peters/Documents/picturesForPaintAppProject/moveCursor.png");
			moveIcon = icon.getImage();
			moveIcon = moveIcon.getScaledInstance(40, 40, Image.SCALE_SMOOTH);
			int y=(int) (s.getShape().getBounds2D().getY() - 40); y=(y<0)?0:y;
				moveButton = new Rectangle2D.Double(
						(int) ((s.getShape().getBounds2D().getX()) + (s.getShape().getBounds2D().getWidth() / 2 - 20)),y, 40, 40);
				int topX=(int) s.getShape().getBounds2D().getX()-10; 
				int topY=(int) s.getShape().getBounds2D().getY()-10;
				int width=(int) s.getShape().getBounds2D().getWidth()+20;
				int height=(int) s.getShape().getBounds2D().getHeight()+20;
			if(!s.getShape().getClass().toString().contains("Line2D")){
			resizeButton = new Rectangle2D.Double(topX,topY,width,height);
			resizeShape=s.getShape();
			}else{
				resizeLine=(Line2D) s.getShape();
				if(resizeLine.getP1().getX()<resizeLine.getP2().getX()){
					point1=resizeLine.getP1();
					point2=resizeLine.getP2();
				}else{
					point1=resizeLine.getP2();
					point2=resizeLine.getP1();
				}
				resizeLine1=new Ellipse2D.Double(point1.getX()-10,point1.getY()-10,20,20);
				resizeLine2=new Ellipse2D.Double(point2.getX()-10,point2.getY()-10,20,20);
				resizeButton=null;
				resizeSelected=false;
			}
		}// End of createShapeOutline
	}// End of image Class
	private image getImagePanel(){
		return imagePanel;
	}
	private int getMementoIndex(){
		return mementoIndex;
	}
	private int getMementoNumber(){
		return mementoNumber;
	}
	private void setMementoIndex(int x){
		mementoIndex+=x;
	}
	private void setMementoNumber(int x){
		mementoNumber+=x;;
	}
	public ArrayList<Memento> getUndoIndexes() {
		return undoIndexes;
	}
	public void setUndoIndexes(ArrayList<Memento> undoIndexes) {
		this.undoIndexes = undoIndexes;
	}
}// End of PaintApp Class

class CareTaker{
	ArrayList<Memento> savedArticles=new ArrayList<Memento>();
	public void addMemento(Memento m){savedArticles.add(m);}
	public Memento getMemento(int index){return savedArticles.get(index);}
	public int getMementoListSize(){
		return savedArticles.size();
	}
	public void removeIndex(Memento index){savedArticles.remove(index);}
	public void removeIndex(int index){savedArticles.remove(index);}
	
}//End of CareTaker Class
class Memento{
	private ArrayList<paintCompoenent> list;
	public Memento(ArrayList<paintCompoenent> list){this.list=list;}
	public ArrayList<paintCompoenent> getSavedArticle(){return list;}
}//End of Memento Class
class Originator{
	private ArrayList<paintCompoenent> list;
	public void set(ArrayList<paintCompoenent> list){
		this.list=list;
	}
	public Memento storeMemento(){
		return new Memento(list);
	}
	public ArrayList<paintCompoenent> restoreFromMemento(Memento m){
		list= m.getSavedArticle();
		return list;
	}
}//End of Originator Class

interface paintCompoenent{
	 Image getImage();
	 void setImage(Image image);
	 float getTransparency();
	 void setTransparency(float transparency);
	 int getX();
	 void setX(int x);
	 int getY();
	 void setY(int y);
	 int getWidth();
	 void setWidth(int width);
	 int getHeight();
	 void setHeight(int height);
	 GradientPaint getPaint();
	 void setPaint(GradientPaint paint);
	 int getThickness();
	 void setThickness(int thickness);
	 Color getFillColor();
	 void setFillColor(Color fillColor);
	 Color getStrokeColor();
	 void setStrokeColor(Color strokeColor);
	 Shape getShape();
	 void setShape(Shape shape);
	 void setTransparency(Float transparency);
}
class imageClass implements paintCompoenent{
	
	private Image image;
	private float transparency;
	private int x,y,width,height;
	public imageClass(Image image, float transparency, int x, int y, int width, int height) {
		this.image = image;
		this.transparency = transparency;
		this.x = x;
		this.y = y;
		this.width = width;
		this.height = height;
	}
	public Image getImage() {
		return image;
	}
	public void setImage(Image image) {
		this.image = image;
	}
	public float getTransparency() {
		return transparency;
	}
	public void setTransparency(float transparency) {
		this.transparency = transparency;
	}
	public int getX() {
		return x;
	}
	public void setX(int x) {
		this.x = x;
	}
	public int getY() {
		return y;
	}
	public void setY(int y) {
		this.y = y;
	}
	public int getWidth() {
		return width;
	}
	public void setWidth(int width) {
		this.width = width;
	}
	public int getHeight() {
		return height;
	}
	public void setHeight(int height) {
		this.height = height;
	}
	public GradientPaint getPaint() {return null;}
	public void setPaint(GradientPaint paint) {}
	public int getThickness() {return 0;}
	public void setThickness(int thickness) {}
	public Color getFillColor() {return null;}
	public void setFillColor(Color fillColor) {}
	public Color getStrokeColor() {return null;}
	public void setStrokeColor(Color strokeColor) {}
	public Shape getShape() {return null;}
	public void setShape(Shape shape) {}
	public void setTransparency(Float transparency) {}
}//End of imageClass
class shapeClass implements paintCompoenent{
	private Shape shape;
	private Float transparency;
	private Color fillColor,strokeColor;
	private int thickness;
	private GradientPaint paint;
	public shapeClass(Shape shape,Float transparency,Color fillColor,Color strokeColor,int thickness,GradientPaint paint){
		this.shape=shape;
		this.transparency=transparency;
		this.fillColor=fillColor;
		this.strokeColor=strokeColor;
		this.thickness=thickness;
		this.paint=paint;
	}
	public GradientPaint getPaint() {
		return paint;
	}
	public void setPaint(GradientPaint paint) {
		this.paint = paint;
	}
	public int getThickness() {
		return thickness;
	}
	public void setThickness(int thickness) {
		this.thickness = thickness;
	}
	public Color getFillColor() {
		return fillColor;
	}
	public void setFillColor(Color fillColor) {
		this.fillColor = fillColor;
	}
	public Color getStrokeColor() {
		return strokeColor;
	}
	public void setStrokeColor(Color strokeColor) {
		this.strokeColor = strokeColor;
	}
	public Shape getShape() {
		return shape;
	}
	public void setShape(Shape shape) {
		this.shape = shape;
	}
	public float getTransparency() {
		return transparency;
	}
	public void setTransparency(Float transparency) {
		this.transparency = transparency;
	}
	public Image getImage() {return null;}
	public void setImage(Image image) {}
	public void setTransparency(float transparency) {}
	public int getX() {return 0;}
	public void setX(int x) {}
	public int getY() {return 0;}
	public void setY(int y) {}
	public int getWidth() {return 0;}
	public void setWidth(int width) {}
	public int getHeight() {return 0;}
	public void setHeight(int height) {}
	
}//End of shapeClass
class ImageTransferable implements Transferable{
    private Image image;

    public ImageTransferable (Image image)
    {
        this.image = image;
    }

    public Object getTransferData(DataFlavor flavor)
        throws UnsupportedFlavorException
    {
        if (isDataFlavorSupported(flavor))
        {
            return image;
        }
        else
        {
            throw new UnsupportedFlavorException(flavor);
        }
    }

    public boolean isDataFlavorSupported (DataFlavor flavor)
    {
        return flavor == DataFlavor.imageFlavor;
    }

    public DataFlavor[] getTransferDataFlavors ()
    {
        return new DataFlavor[] { DataFlavor.imageFlavor };
    }
}//End of ImageTransferable